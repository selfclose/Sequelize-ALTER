# Sequelize-ALTER
When your developing your app with sequelize, No need to {force: true} for destroy your data, Just simple hook to put in sequelize for ALTER table

* Auto Add-Remove Table when it's new or no longer anymore

Sequelize
Just add this hook and you ready to go

When you starting define 

```javascript
var Sequelize = require('sequelize');

var sequelize = new Sequelize('my-db', 'my-username', 'my-password', {
    host: 'localhost',
    dialect: 'mysql',
    logging: true, //Show logging on console?

    pool: {
        max: 5,
        min: 0,
        idle: 10000
    },
    
    //Add this hook below
    define: {
        hooks: {
            beforeSync: function (options) {
                var self = this
                    , attributes = this.tableAttributes;
                    if (options.alter) {
                        return self.QueryInterface.describeTable(self.getTableName(options))
                            .then(function (columns) {
                                var changes = []; // array of promises to run
                                _.each(attributes, function (columnDesc, columnName) {
                                    //add new column if not exist
                                    if (!columns[columnName]) {
                                        changes.push(self.QueryInterface.addColumn(self.getTableName(options), columnName, attributes[columnName]));
                                    }
                                });
                                _.each(columns, function (columnDesc, columnName) {
                                    if (!attributes[columnName]) {
                                        //delete column if no longer
                                        changes.push(self.QueryInterface.removeColumn(self.getTableName(options), columnName, options));
                                    } else {
                                        //still bug with primary key, unique etc. Don't use it for now.
                                        // if (!attributes[columnName].primaryKey) {
                                        //     changes.push(self.QueryInterface.changeColumn(self.getTableName(options), columnName, attributes[columnName]));
                                        // }
                                    }
                                });
                                return Promise.all(changes);
                            });
                    }

            }
        }
    }
});
```

Usage:

```javascript
sequelize.sync({alter: true});
```
