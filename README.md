# Sequelize-ALTER
When you are developing your app with sequelize, Some time need to add/remove column just for a little change, Why I need to wipe all data with `{force: true}`?.

Now just add simple hook to help you Auto Add-Remove column when it's new or no longer anymore

_(I'm not own this script, The credit goes [here](https://github.com/meyer9/sequelize/commit/5945d1087a81c4fcbd1a819c654e5064c13a1ef2))_, I made it work on hooking ('cuz I can't wait for an update), 

**(No script for download... just copy it...)**

```javascript
var Sequelize = require('sequelize');
var lod = require('lodash'); //Already come with Sequelize

var sequelize = new Sequelize('my-db', 'my-username', 'my-password', {
    host: 'localhost',
    dialect: 'mysql',

    pool: {
        max: 5,
        min: 0,
        idle: 10000
    },
    
    //----- Copy this hook below -----//
    define: {
        hooks: {
            beforeSync: function (options) {
                var self = this, attributes = this.tableAttributes;
                if (options.alter) {
                    return self.QueryInterface.describeTable(self.getTableName(options)).then(function (columns) {
                        var changes = []; // array of promises to run
                        lod.each(attributes, function (columnDesc, columnName) {
                            //add new column if not exist
                            if (!columns[columnName]) {
                                console.log('add new column: '+columnName);
                                changes.push(self.QueryInterface.addColumn(self.getTableName(options), columnName, attributes[columnName]));
                            }
                        });
                        lod.each(columns, function (columnDesc, columnName) {
                            //delete column if no longer
                            if (!attributes[columnName]) {
                                console.log('remove column: '+columnName);
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
