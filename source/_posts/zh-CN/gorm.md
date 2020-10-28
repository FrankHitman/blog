---
title: gorm的使用
lang: zh-CN
date: 2019-12-09 17:16:01
categories:
    - golang
tags: 
    - gorm
    - orm
---

### ForeignKey
gorm model struct中定义外键字段后，如果dialect是mysql，那么使用`AutoMigrate`生成表时候不会自动生成外键。参考见[GitHub issue](https://github.com/jinzhu/gorm/issues/450)
<!--more-->
```go
type ChargingHistory struct {
	Id                string          `gorm:"type:varchar(36);primary_key" json:"id"`
	ElectricVehicleId string          `gorm:"type:varchar(36)" json:"electric_vehicle_id"`
	ElectricVehicle   ElectricVehicle `gorm:"foreignkey:ElectricVehicleId" json:"electric_vehicle"`
	ChargerId         string          `gorm:"type:varchar(36)" json:"charger_id"`
	Charger           Charger         `gorm:"foreignkey:ChargerId" json:"charger"`
	MobileUser        MobileUser      `gorm:"foreignkey:MobileUserId" json:"-"`
	MobileUserId      string          `gorm:"type:varchar(36)" json:"mobile_user_id"`
}
```
所以如果需要使用model生成外键，候需要使用`AddForeignKey`语句生成外键
```go
db.AutoMigrate(
    &model.MobileUser{},
    &model.Charger{},
    &model.ElectricVehicle{},

    &model.ChargingHistory{},
)

db.AutoMigrate(&model.ChargingHistory{}).
    AddForeignKey("charger_id", "charger(id)", "restrict", "restrict")
```

### Query
使用First,Find查询默认会根据主键查询，另外如果有deleted_at字段，会默认过滤掉deleted_at不为空的记录
```go
func (u *MobileUser) Get(db igorm.Gormw) (*MobileUser, error) {
	if u.Mobile != "" {
		usr := &MobileUser{}
		d := db.Where("mobile = ?", u.Mobile).First(&usr)
		return usr, d.Error()
	}
	// default deleted_at is null and will check id = xxxx if id is not default value
	d := db.First(&u)
	return u, d.Error()
}
```
当使用First查询数据库时候，如果主键为空，那么就取全部数据中的第一条记录返回。
```go
func (e *ElectricVehicle) Get(db igorm.Gormw) (*ElectricVehicle, error) {
	// when e.id == "" db.First() will get the first item which order by id asc
	// when e.id != "" db.First() will get the first item which filter by id = xxx order by id asc
	d := db.First(&e)
	return e, d.Error()
}
```

### 定义与生成联合索引
使用struct定义联合索引时候，需要注意联合索引的先后排序顺序，因此需要把主排序索引列放到model struct结构体中其他次要排序索引列的前面。
例如如下idx_user_charger联合索引把MobileUserId列声明在ChargerId列前面。然后在AutoMigrate时候gorm会帮助自动生成联合索引
```go
type PrivateCharger struct {
	Base
	MobileUserId string     `gorm:"type:varchar(36);not null;index:idx_user_charger" json:"mobile_user_id"`
	MobileUser   MobileUser `gorm:"foreignkey:MobileUserId" json:"mobile_user"`
	ChargerId    string     `gorm:"type:varchar(36);not null;index:idx_user_charger" json:"charger_id"`
	Charger      Charger    `gorm:"foreignkey:ChargerId" json:"charger"`
	IsOwner      bool       `gorm:"type:tinyint(1);default:0;comment:'isowner(1) isnotowner(0)'" json:"is_owner"`
	DeletedAt    *time.Time `json:"deleted_at"`
	IsShared     bool       `gorm:"default:0;comment:'unshared(0) shared(1)'" json:"is_shared"`
}
```

### Save
在Gorm中使用Save时候，如果没有主键字段，那么会创建一条新纪录，依赖于数据库自动生成主键；如果有主键字段，那么会经过update -> select -> insert三个阶段。
先是尝试着根据条件更新数据库记录，如果更新失败那么再执行`FirstOrCreate`方法；FirstOrCreate就是先根据条件查询，如果存在返回数据库中记录，如果不存在，就创建一条新纪录。

我们系统中主键id是由程序生成，而不是数据库自动创建的。以下方法用于设置默认车的功能，在新建和修改的时候都可以使用。
```go
func (e *ElectricVehicle) SetDefault(db igorm.Gormw, userId string) error {
	tx := db.Begin()

	if err := tx.Model(&ElectricVehicle{}).Where("mobile_user_id = ?", userId).Update("is_default", 0).Error(); err != nil {
		tx.Rollback()
		return err
	}

	// when used in create e this will go pass update -> select -> insert 3 progress. =_=
	if err := tx.Save(e).Error(); err != nil {
		tx.Rollback()
		return err
	}

	tx.Commit()
	return nil
}
```

