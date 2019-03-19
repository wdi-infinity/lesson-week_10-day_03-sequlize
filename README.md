# Learning Sequlize with Manal's Potion Shop !

![sequlize](https://i.ytimg.com/vi/a4dW7flCsy4/maxresdefault.jpg)

# Agenda

1. Before we begin...
2. Project Setup
3. Sequelize Setup
4. Generating Models
5. Migrations
6. Assocation
7. Seeds
8. CRUD Operation for Customers, Potions, Order and Categories

# Before we Begin

We're going to be using PostgreSQL and Sequelize as the ORM of choice to write a minimalistic Potion Shop application.

Manal known as a witch which is a fictional character who appears in Masaqil, she likes to mix ingredients for several potions including the True Love Potion! she is now having a lot of customers and needs help to build a database to keep up with her business

# The models

1. Customer
2. Order
3. Potion
4. Category

![models](https://i.imgur.com/kLq0MBQ.jpg)

# Project Setup

```
mkdir sequlize-node
cd sequlize-node
mkdir db
npm init -y
```

# Sequelize Setup

```
npm install -g sequelize-cli
```

This will allow us to use the sequelize as a command line tool that helps us to create and manage our sequelize files.

In addition, you will need to also install sequelize module localy in order to utilize the command line tool.

```
npm install --save sequelize pg pg-hstore path
```

## create

```
touch .sequelizerc
sequelize init
```

This will generate a few files for you, your project folder should now look like this:

```
.
├── config
│ └── config.json
├── migrations
├── models
│ └── index.js
├── seeders
```

## Configuring your database

The generated config/config.json file begins with an environment level. This let's you configure different configurations for different environments

Edit your development settings in config/config.json to point to your postgres database.

```
// db/config/config.json

{
  "development": {
  "username": <your username>,
  "password": null,
  "database": "sequelize_demo",
  "host": "127.0.0.1",
  "dialect": "postgres",
  "define": {
        "underscored": true
    }
  },
...
}
```

# Generating Models

```
sequelize model:create --name Customer --attributes first_name:string,last_name:string

sequelize model:create --name Potion --attributes name:string,description:string

sequelize model:create --name Category --attributes name:string
```

http://sequelize.readthedocs.org/en/latest/docs/migrations/

# Migrations - Oops forgot something!

we forgot to add the price in the Potion Model !!
let add it with this commmand

```
sequelize migration:create --name add-price-to-potion
```

This time you run the command, it only generates a migration file for you. When you open it, you notice it's pretty bare!

migrations/-add-price-to-potion.js

```js
"use strict";

module.exports = {
  return up: (queryInterface, Sequelize) => {
    /*
      Add altering commands here.
      Return a promise to correctly handle asynchronicity.

      Example:
      return queryInterface.createTable('users', { id: Sequelize.INTEGER });
    */
  },

  return down: (queryInterface, Sequelize) => {
    /*
      Add reverting commands here.
      Return a promise to correctly handle asynchronicity.

      Example:
      return queryInterface.dropTable('users');
    */
  }
};
```

But, because you're awesome and read the documentation, you know sequelize migrations has an addColumn function that will fix the problem for you!

After filling in the up and down functions, your migration file looks like this:

```js
"use strict";
module.exports = {
  return up: function(queryInterface, Sequelize) {
    queryInterface.addColumn("Potions", "price", Sequelize.DOUBLE);
  },

  return down: function(queryInterface, Sequelize) {
    queryInterface.removeColumn("Potions", "price");
  }
};
```

It's important to always have both up and down methods in your migration script. Every time you run the following commnd it will call the up method.

```
sequelize db:migrate
```

And if you ever want to revert backwards which will call the down method. you can call

```
sequelize db:migrate:undo
```

Now that you fixed your migrations, you need to update your model to reflect the changes.

```js
//models/potion.js

"use strict";
module.exports = (sequelize, DataTypes) => {
  const Potion = sequelize.define(
    "Potion",
    {
      name: DataTypes.STRING,
      description: DataTypes.STRING,
      price: DataTypes.DOUBLE
    },
    {}
  );
  Potion.associate = function(models) {
    // associations can be defined here
  };
  return Potion;
};
```

Now if you run migrations, it will update your table to have price as well!

<hr>

## other examples

### Rename Table

```js
queryInterface.renameTable("nameOfTheExistingTable", "newNameOfThTable");
```

### Drop Table

```js
queryInterface.dropTable("nameOfTheExistingTable");
```

### Change Column

```js
queryInterface.changeColumn(
  "nameOfAnExistingTable",
  "nameOfAnExistingAttribute",
  {
    type: Sequelize.FLOAT,
    allowNull: false,
    defaultValue: 0.0
  }
);
```

### Rename Column

```js
queryInterface.renameColumn(
  "nameOfTheExistingTable",
  "nameOfAnExistingColumn",
  "newColumnName"
);
```

## migrations commands

<table>
  <tr>
    <td>sequelize db:migrate</td>
    <td>Run pending migrations</td>
  </tr>

  <tr>
    <td>sequelize db:migrate:schema:timestamps:add </td>
    <td>Update migration table to have timestamps</td>
  </tr>

  <tr>
    <td>sequelize db:migrate:status</td>
    <td>Reverts a migration</td>
  </tr>

  <tr>
    <td>sequelize db:migrate:undo:all</td>
    <td> Revert all migrations</td>
  </tr>

</table>

# Assocation

## one to many

### - Potion Model

```js
Potion.belongsTo(models.Category, {
  foreignKey: "category_Id",
  onDelete: "CASCADE"
});
```

### - Category Model

```js
Category.hasMany(models.Potion, {
  foreignKey: "category_Id",
  as: "potions"
});
```

## many to many

### - Customer Model

```js
const Order = sequelize.define("Orders", {
  total: DataTypes.DOUBLE,
  count: DataTypes.INTEGER
});

Customer.belongsToMany(models.Potion, {
  through: orders,
  as: "groupMemberships",
  foreignKey: {
    field: "customer_id"
  }
});
```

# Seeding your database

When developing databases with a team, it can be important that everyone is working with the same data. Or you might have information that you want to enter in your database initally, like admin accounts or something like that. You can do this with Seeders.

```
sequelize seed:create --name customers
sequelize seed:create --name categories
sequelize seed:create --name potions
sequelize seed:create --name orders
```

After filling in the up and down functions, your seed file looks like this:

```js
"use strict";

module.exports = {
  return up: (queryInterface, Sequelize) => {
    queryInterface.bulkInsert(
      "Categories",
      [
        {
          name: "good",
          updated_at: new Date(),
          created_at: new Date()
        },
        {
          name: "evil",
          updated_at: new Date(),
          created_at: new Date()
        },
        {
          name: "forbidden",
          updated_at: new Date(),
          created_at: new Date()
        }
      ],
      {}
    );
  },

  return down: (queryInterface, Sequelize) => {
    return queryInterface.bulkDelete("Categories", null, {});
  }
};
```

## seed commands

<table>
  <tr>
    <td> sequelize db:seed</td>
    <td>Run specified seeder</td>
  </tr>

  <tr>
    <td>sequelize db:seed:undo</td>
    <td>Deletes data from the database</td>
  </tr>

  <tr>
    <td>sequelize db:seed:all</td>
    <td>Run every seeder</td>
  </tr>

  <tr>
    <td>sequelize db:seed:undo:all </td>
    <td>Deletes data from the database</td>
  </tr>

</table>

# CRUD Operation

```js
const db = require("./db/models");

// Create

db.Customer.create({ first_name: "ghadeer", last_name: "alkhathlan" })
  .then(customer => console.log(customer.get({ plain: true })))
  .catch(e => console.log(e));

// Read
db.Customer.findAll()
  .then(customers =>
    console.log(customers.map(customer => customer.get({ plain: true })))
  )
  .catch(e => console.log(e));

db.Customer.findByPk(1)
  .then(customer => console.log(customer.get({ plain: true })))
  .catch(e => console.log(e));

db.Customer.findOne({ where: { first_name: "ghadeer" } })
  .then(customer => console.log(customer.get({ plain: true })))
  .catch(e => console.log(e));

// Update
db.Customer.findOne({ where: { first_name: "ghadeer" } })
  .then(customer => {
    customer
      .update({ first_name: "sara" })
      .then(customer => console.log(customer.get({ plain: true })))
      .catch(e => console.log(e));
  })
  .catch(e => console.log(e));

// Delete
db.Customer.findOne({ where: { first_name: "ghadeer" } })
  .then(customer => {
    customer
      .destroy()
      .then(() => console.log("success")))
      .catch(e => console.log(e));
  })
  .catch(e => console.log(e));
```

# Using Include

# Assocations Methods

# Reading

[Documintaion](http://docs.sequelizejs.com/manual/installation/getting-started.html)

[Validations](http://docs.sequelizejs.com/en/latest/docs/models-definition/#validations)

[Assocation](http://docs.sequelizejs.com/manual/associations.html)
