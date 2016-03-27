# qml-extra-models
This repo includes models that are not included in QML directly.

## SQL
----
#### SqlModel
The SqlModel is a subclassed QSqlTableModel, that is created entirely from QML. This model lives on the UI thread.
```qml
    import Models 1.0

    ListView {
        id: listView;
        
        model: SqlModel {
            id: sqlModel;
            tableName: "Movies";
            autoCreate: true;
            connectionName: "LIBRARY";
            fileLocation: "/home/username/Desktop/db.sqlite";
            
            SqlColumn { name: "Title"; type: "TEXT"; }
            SqlColumn { name: "Poster"; type: "TEXT"; }
            SqlColumn { name: "Description"; type: "TEXT"; }
            SqlColumn { name: "AbsoluteFilePath"; type: "TEXT NOT NULL UNIQUE"; }
            
            Component.onCompleted: { sqlModel.finishModelConstruction(); }
        }
        
        delegate: Item {
            width: 100;
            height: 50;
            
            Text { text: Title; }
        }
    }
```

#### C++ slot definition
```c++
    // These are all member variables of the SqlModel. The column is the 
    //  SqlColumn::name Q_PROPERTY.

    // Sets an SQL filter on the query. The added filter will remain until
    // 'clearFilter()' is called on the filtered column. 
    void setFilter( const QString column
                    , const QVariant value
                    , const FilterType type = FilterType::Like );

    // Removes an SQL filter on the query.
    void clearFilter( const QString column );

    // This function needs to be called once all the Q_PROPERTIES have been assigned.
    // This is called in Component.OnCompleted and "MUST" be called from there.
    void finishModelConstruction();

    // Inserts a new row into the sql model.
    bool addRow( const QVariantMap rowData );

    //Remove row from sql model.
    bool deleteRow( int index, const QString column, const QVariant absFilePath );

    // Updates the row with the new value;
    bool updateRow( int index, const QString column, const QVariant oldData, const QVariant newData );

    // Deletes the database and resets the sql schema. The model is now initialized
    // but is still empty of rows.
    void clearDatabase();
```

##### QML slot usage
```qml
// Inside ListView delegate
// ...
    sqlModel.deleteRow( index, "AbsoluteFilePath", AbsoluteFilePath )
    sqlModel.updateRow( index, "AbsoluteFilePath", AbsoluteFilePath, "/home/lee/file" + Math.random() + ".png" );
    sqlModel.setFilter( "Title", "zelda" );
    sqlModel.clearFilter( "Title", "mario" );
    sqlModel.clearDatabase();
```

### SqlThreadedModel
SqlThreadedModel is a subclass of QAbstractTableModel and serves as a proxy model
for SqlModel, which is kept as a member variable internally. The internal SqlModel is offloaded onto a separate thread, and queries are sent via signals.

This model will not hang the UI thread, unlike SqlModel, which potentially could if the database queries thousands of results. Usage is identical to SqlModel, except the return variables attached to the slots will always return true and are not guaranteed to finish when the function returns. I will attach signals soon.



