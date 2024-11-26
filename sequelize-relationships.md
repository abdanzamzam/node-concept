
# Sequelize Relationships: 1:1, 1:N, and N:N

This document explains how to set up and use relationships in Sequelize, including one-to-one (1:1), one-to-many (1:N), and many-to-many (N:N).

## **1. One-to-One (1:1)**

### **Explanation**
A one-to-one relationship means one entity is related to exactly one other entity.

### **Example:**
A `User` has one `Profile`.

#### **Model: User (`models/user.js`)**
```javascript
User.hasOne(models.Profile, {
  foreignKey: 'userId', // Foreign key in the Profile table
  as: 'profile', // Alias for the relationship
});
```

#### **Model: Profile (`models/profile.js`)**
```javascript
Profile.belongsTo(models.User, {
  foreignKey: 'userId', // Foreign key in the Profile table
  as: 'user', // Alias for the relationship
});
```

---

## **2. One-to-Many (1:N)**

### **Explanation**
A one-to-many relationship means one entity is related to multiple entities.

### **Example:**
A `User` has many `Posts`.

#### **Model: User (`models/user.js`)**
```javascript
User.hasMany(models.Post, {
  foreignKey: 'userId', // Foreign key in the Post table
  as: 'posts', // Alias for the relationship
});
```

#### **Model: Post (`models/post.js`)**
```javascript
Post.belongsTo(models.User, {
  foreignKey: 'userId', // Foreign key in the Post table
  as: 'user', // Alias for the relationship
});
```

---

## **3. Many-to-Many (N:N)**

### **Explanation**
A many-to-many relationship means multiple entities in one table are related to multiple entities in another table.

### **Example:**
A `User` can belong to multiple `Groups`, and a `Group` can have multiple `Users`.

#### **Model: User (`models/user.js`)**
```javascript
User.belongsToMany(models.Group, {
  through: 'UserGroups', // Join table name
  foreignKey: 'userId',
  otherKey: 'groupId',
  as: 'groups',
});
```

#### **Model: Group (`models/group.js`)**
```javascript
Group.belongsToMany(models.User, {
  through: 'UserGroups', // Join table name
  foreignKey: 'groupId',
  otherKey: 'userId',
  as: 'members',
});
```

#### **Join Table: UserGroups (`migrations/create-user-groups.js`)**
```javascript
'use strict';
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.createTable('UserGroups', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER,
      },
      userId: {
        type: Sequelize.INTEGER,
        references: {
          model: 'Users',
          key: 'id',
        },
        onUpdate: 'CASCADE',
        onDelete: 'CASCADE',
      },
      groupId: {
        type: Sequelize.INTEGER,
        references: {
          model: 'Groups',
          key: 'id',
        },
        onUpdate: 'CASCADE',
        onDelete: 'CASCADE',
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE,
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE,
      },
    });
  },
  async down(queryInterface, Sequelize) {
    await queryInterface.dropTable('UserGroups');
  },
};
```

---

## **4. Adding Associations in `models/index.js`**
Add the following lines to `models/index.js` after all models are loaded:
```javascript
models.User.associate(models);
models.Profile.associate(models);
models.Post.associate(models);
models.Group.associate(models);
```

---

## **5. Usage Examples**

### **1:1 Relationship**
Retrieve a `User` along with their `Profile`:
```javascript
const user = await User.findByPk(1, { include: { model: Profile, as: 'profile' } });
console.log(user.profile);
```

### **1:N Relationship**
Retrieve a `User` along with their `Posts`:
```javascript
const user = await User.findByPk(1, { include: { model: Post, as: 'posts' } });
console.log(user.posts);
```

### **N:N Relationship**
Retrieve a `User` along with their `Groups`:
```javascript
const user = await User.findByPk(1, { include: { model: Group, as: 'groups' } });
console.log(user.groups);
```

---

With these configurations, you can efficiently manage relationships in Sequelize, including one-to-one, one-to-many, and many-to-many associations.
