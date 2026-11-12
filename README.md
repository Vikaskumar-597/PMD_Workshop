# Country and Currency Management App 📱

## 📌 Project Description

This is an Android application developed using Java, XML, and SQLite Database. The application stores country names and their respective currencies and displays them in a ListView.

It supports all four CRUD operations (Create, Read, Update, Delete), allowing users to add, view, modify, and delete country records.

## 🎯 Objectives

- Create country records using SQLite Database.
- Display country names and currencies using ListView.
- Add new country and currency details.
- Modify existing country records.
- Update and delete database records.

## 🛠️ Technologies Used

- Programming Language: Java
- Frontend: XML
- Database: SQLite
- IDE: Android Studio
- Platform: Android

## 📂 Project Structure

```text
pmdworkshopApp/
│
├── app/src/main/
│   ├── java/com/example/pmdworkshop/
│   │   ├── DatabaseHelper.java
│   │   ├── DBManager.java
│   │   ├── CountryListActivity.java
│   │   ├── AddCountryActivity.java
│   │   └── ModifyCountryActivity.java
│   │
│   ├── res/layout/
│   │   ├── activity_country_list.xml
│   │   ├── activity_add_country.xml
│   │   └── activity_modify_country.xml
│   │
│   └── AndroidManifest.xml
│
└── README.md
```

# 💻 Source Code

## 1. DatabaseHelper.java

Creates the SQLite database and Country table.

```java
package com.example.pmdworkshop;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

public class DatabaseHelper extends SQLiteOpenHelper {

    public static final String DB_NAME = "CountryDB";
    public static final String TABLE_NAME = "Country";

    public static final String ID = "id";
    public static final String COUNTRY = "country";
    public static final String CURRENCY = "currency";

    public DatabaseHelper(Context context) {
        super(context, DB_NAME, null, 1);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        String query = "CREATE TABLE Country (" +
                "id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                "country TEXT, currency TEXT)";
        db.execSQL(query);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db,
                          int oldVersion, int newVersion) {
        db.execSQL("DROP TABLE IF EXISTS Country");
        onCreate(db);
    }
}
```

## 2. DBManager.java

Performs Insert, Read, Update, and Delete operations.

```java
package com.example.pmdworkshop;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;

public class DBManager {

    DatabaseHelper helper;
    SQLiteDatabase db;

    public DBManager(Context context) {
        helper = new DatabaseHelper(context);
    }

    public void open() {
        db = helper.getWritableDatabase();
    }

    // Insert
    public boolean insert(String country, String currency) {
        ContentValues values = new ContentValues();
        values.put("country", country);
        values.put("currency", currency);

        return db.insert("Country", null, values) != -1;
    }

    // Read
    public Cursor getAllCountries() {
        return db.rawQuery(
                "SELECT id AS _id, country, currency FROM Country",
                null);
    }

    // Read selected country
    public Cursor getCountry(int id) {
        return db.rawQuery(
                "SELECT * FROM Country WHERE id=?",
                new String[]{String.valueOf(id)});
    }

    // Update
    public boolean update(int id, String country, String currency) {
        ContentValues values = new ContentValues();
        values.put("country", country);
        values.put("currency", currency);

        return db.update("Country", values, "id=?",
                new String[]{String.valueOf(id)}) > 0;
    }

    // Delete
    public boolean delete(int id) {
        return db.delete("Country", "id=?",
                new String[]{String.valueOf(id)}) > 0;
    }

    public void close() {
        helper.close();
    }
}
```

## 3. CountryListActivity.java

Displays all country records in a ListView. Click a record to modify it and long-press a record to delete it.

```java
package com.example.pmdworkshop;

import android.content.Intent;
import android.database.Cursor;
import android.os.Bundle;
import android.widget.*;
import androidx.appcompat.app.AppCompatActivity;
import java.util.ArrayList;

public class CountryListActivity extends AppCompatActivity {

    ListView listView;
    Button btnAdd;
    DBManager manager;

    ArrayList<String> countryList;
    ArrayList<Integer> idList;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_country_list);

        listView = findViewById(R.id.listView);
        btnAdd = findViewById(R.id.btnAdd);

        manager = new DBManager(this);
        manager.open();

        countryList = new ArrayList<>();
        idList = new ArrayList<>();

        btnAdd.setOnClickListener(v -> {
            startActivity(new Intent(this,
                    AddCountryActivity.class));
        });

        listView.setOnItemClickListener((parent, view,
                position, id) -> {
            Intent intent = new Intent(this,
                    ModifyCountryActivity.class);
            intent.putExtra("id", idList.get(position));
            startActivity(intent);
        });

        listView.setOnItemLongClickListener(
                (parent, view, position, id) -> {
            if (manager.delete(idList.get(position))) {
                Toast.makeText(this, "Record Deleted",
                        Toast.LENGTH_SHORT).show();
                loadCountries();
            }
            return true;
        });
    }

    @Override
    protected void onResume() {
        super.onResume();
        loadCountries();
    }

    private void loadCountries() {
        countryList.clear();
        idList.clear();

        Cursor cursor = manager.getAllCountries();

        while (cursor.moveToNext()) {
            int id = cursor.getInt(0);
            String country = cursor.getString(1);
            String currency = cursor.getString(2);

            idList.add(id);
            countryList.add(country + " - " + currency);
        }

        cursor.close();

        ArrayAdapter<String> adapter = new ArrayAdapter<>(
                this,
                android.R.layout.simple_list_item_1,
                countryList);

        listView.setAdapter(adapter);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        manager.close();
    }
}
```

## 4. AddCountryActivity.java

Allows users to insert a new country and its currency.

```java
package com.example.pmdworkshop;

import android.os.Bundle;
import android.widget.*;
import androidx.appcompat.app.AppCompatActivity;

public class AddCountryActivity extends AppCompatActivity {

    EditText etCountry, etCurrency;
    Button btnSave;
    DBManager manager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_add_country);

        etCountry = findViewById(R.id.etCountry);
        etCurrency = findViewById(R.id.etCurrency);
        btnSave = findViewById(R.id.btnSave);

        manager = new DBManager(this);
        manager.open();

        btnSave.setOnClickListener(v -> {
            String country = etCountry.getText().toString().trim();
            String currency = etCurrency.getText().toString().trim();

            if (country.isEmpty() || currency.isEmpty()) {
                Toast.makeText(this, "Enter all details",
                        Toast.LENGTH_SHORT).show();
            } else {
                if (manager.insert(country, currency)) {
                    Toast.makeText(this, "Country Added",
                            Toast.LENGTH_SHORT).show();
                    finish();
                }
            }
        });
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        manager.close();
    }
}
```

## 5. ModifyCountryActivity.java

Allows users to retrieve, update, and delete selected country records.

```java
package com.example.pmdworkshop;

import android.database.Cursor;
import android.os.Bundle;
import android.widget.*;
import androidx.appcompat.app.AppCompatActivity;

public class ModifyCountryActivity extends AppCompatActivity {

    EditText etCountry, etCurrency;
    Button btnUpdate, btnDelete;
    DBManager manager;
    int countryId;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_modify_country);

        etCountry = findViewById(R.id.etCountry);
        etCurrency = findViewById(R.id.etCurrency);
        btnUpdate = findViewById(R.id.btnUpdate);
        btnDelete = findViewById(R.id.btnDelete);

        manager = new DBManager(this);
        manager.open();

        countryId = getIntent().getIntExtra("id", -1);

        Cursor cursor = manager.getCountry(countryId);

        if (cursor.moveToFirst()) {
            etCountry.setText(cursor.getString(1));
            etCurrency.setText(cursor.getString(2));
        }
        cursor.close();

        btnUpdate.setOnClickListener(v -> {
            String country = etCountry.getText().toString().trim();
            String currency = etCurrency.getText().toString().trim();

            if (country.isEmpty() || currency.isEmpty()) {
                Toast.makeText(this, "Enter all details",
                        Toast.LENGTH_SHORT).show();
            } else if (manager.update(countryId, country, currency)) {
                Toast.makeText(this, "Record Updated",
                        Toast.LENGTH_SHORT).show();
                finish();
            }
        });

        btnDelete.setOnClickListener(v -> {
            if (manager.delete(countryId)) {
                Toast.makeText(this, "Record Deleted",
                        Toast.LENGTH_SHORT).show();
                finish();
            }
        });
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        manager.close();
    }
}
```

# 🎨 XML Layout Files

## 1. activity_country_list.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="20dp">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Country and Currency Details"
        android:textSize="23sp"
        android:textStyle="bold"
        android:gravity="center"
        android:padding="15dp"/>

    <Button
        android:id="@+id/btnAdd"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Add Country"/>

    <ListView
        android:id="@+id/listView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</LinearLayout>
```

## 2. activity_add_country.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="25dp"
    android:gravity="center">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Add Country"
        android:textSize="25sp"
        android:textStyle="bold"
        android:gravity="center"/>

    <EditText
        android:id="@+id/etCountry"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Country Name"/>

    <EditText
        android:id="@+id/etCurrency"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Currency"/>

    <Button
        android:id="@+id/btnSave"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Save Country"/>

</LinearLayout>
```

## 3. activity_modify_country.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="25dp"
    android:gravity="center">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Modify Country"
        android:textSize="25sp"
        android:textStyle="bold"
        android:gravity="center"/>

    <EditText
        android:id="@+id/etCountry"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Country Name"/>

    <EditText
        android:id="@+id/etCurrency"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Currency"/>

    <Button
        android:id="@+id/btnUpdate"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Update"/>

    <Button
        android:id="@+id/btnDelete"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Delete"/>

</LinearLayout>
```

# 📄 AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:allowBackup="true"
        android:label="Country Currency"
        android:supportsRtl="true"
        android:theme="@style/Theme.pmdworkshop">

        <activity
            android:name=".ModifyCountryActivity"
            android:exported="false"/>

        <activity
            android:name=".AddCountryActivity"
            android:exported="false"/>

        <activity
            android:name=".CountryListActivity"
            android:exported="true">

            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>

        </activity>

    </application>
</manifest>
```

# 📱 Application Output

<img width="1325" height="737" alt="Screenshot 2026-11-12 194055" src="https://github.com/user-attachments/assets/b9cae013-9fb9-4cdb-9bbb-141fa1eb9c55" />
<img width="1325" height="738" alt="Screenshot 2026-11-12 194510" src="https://github.com/user-attachments/assets/6ab47564-ccfe-480a-a0b3-143b362a0894" />
<img width="313" height="618" alt="Screenshot 2026-11-12 194109" src="https://github.com/user-attachments/assets/68f3f959-cdbd-4bf3-806a-7d670d7968b4" />
<img width="341" height="617" alt="Screenshot 2026-11-12 194122" src="https://github.com/user-attachments/assets/24cd4340-423c-49b4-96a4-44f568edc6e3" />
<img width="323" height="620" alt="Screenshot 2026-11-12 194439" src="https://github.com/user-attachments/assets/04a07ecc-ce8f-4379-b35b-e559a6014a14" />
<img width="302" height="591" alt="Screenshot 2026-11-12 194454" src="https://github.com/user-attachments/assets/2f473469-2601-4ce9-a14b-8700efec3c83" />
