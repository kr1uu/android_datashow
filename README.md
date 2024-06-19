# android_datashow
---------------------Adapter----------------------------------    
package com.PhanVykiet.adapter;  

import android.content.Context;  
import android.view.LayoutInflater;  
import android.view.View;  
import android.view.ViewGroup;  
import android.widget.BaseAdapter;  
import android.widget.ImageView;  
import android.widget.TextView;  

import com.PhanVykiet.model.Tour;  
import com.PhanVykiet.sqlitetest.MainActivity;  
import com.PhanVykiet.sqlitetest.R;  

import java.util.List;

public class TourAdapter extends BaseAdapter {  
    MainActivity context;  
    int item_layout;  
    List<Tour> Tour;  

    public TourAdapter(MainActivity context, int item_layout, List<Tour> Tour) {  
        this.context = context;  
        this.item_layout = item_layout;  
        this.Tour = Tour;  
    }

    @Override
    public int getCount() {
        return Tour.size();
    }

    @Override
    public Object getItem(int position) {
        return Tour.get(position);
    }

    @Override
    public long getItemId(int position) {
        return 0;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {  
        ViewHolder holder;
        if (convertView == null){
            holder = new ViewHolder();
            LayoutInflater inflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
            convertView = inflater.inflate(item_layout, null);

            holder.txtInfo = convertView.findViewById(R.id.txtInfo);
            holder.imgEdit = convertView.findViewById(R.id.imgEdit);
            holder.imgDelete = convertView.findViewById(R.id.imgDelete);

            convertView.setTag(holder);
        } else {
            holder = (ViewHolder) convertView.getTag();
        }
        Tour p = Tour.get(position);
        holder.txtInfo.setText(p.getTourName()+" - "+String.format("%.0fd", p.getTourPrice()));
        holder.imgEdit.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                context.openEditDialog(p);
            }
        });
        holder.imgDelete.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
            context.openDeleteConfirmDialog(p);
            }
        });

        return convertView;
    }

    public static class ViewHolder{
        TextView txtInfo;
        ImageView imgEdit, imgDelete;
    }
}

----------------------------endAdapter-------------------------------------  
----------------------------Database---------------------------------------  
package com.PhanVykiet.model;

public class Tour {  
    int TourCode;  
    String TourName;  
    double TourPrice;  

    public Tour(int TourCode, String TourName, double TourPrice) {  
        this.TourCode = TourCode;  
        this.TourName = TourName;  
        this.TourPrice = TourPrice;  
    }

    public int getTourCode() { return TourCode; }

    public void setTourCode(int tourCodeCode) {
        this.TourCode = TourCode;
    }

    public String getTourName() {
        return TourName;
    }

    public void setTourName(String TourName) {
        this.TourName = TourName;
    }

    public double getTourPrice() {
        return TourPrice;
    }

    public void setTourPrice(double TourPrice) {
        this.TourPrice = TourPrice;
    }
}

---------------------------------------endDatabase----------------------------------  
---------------------------------------ClassTour----------------------------------  
package com.PhanVykiet.sqlitetest;

import android.content.Context;  
import android.database.Cursor;  
import android.database.sqlite.SQLiteDatabase;  
import android.database.sqlite.SQLiteOpenHelper;  
import android.util.Log;  

import androidx.annotation.Nullable;

public class Database extends SQLiteOpenHelper {  
    public static final String DB_NAME = "tour.sqlite";  
    public static final int DB_VERSION = 1;  
    public static final String TBL_NAME = "Tour";  
    public static final String COL_CODE = "TuorCode";  
    public static final String COL_NAME = "TourName";  
    public static final String COL_PRICE = "TourPrice";  

    public static final String COL_IMG ="TuorImg";

    public Database(@Nullable Context context) {
        super(context, DB_NAME, null, DB_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        String sql = "CREATE TABLE IF NOT EXISTS " + TBL_NAME + " (" + COL_CODE+" INTEGER PRIMARY KEY AUTOINCREMENT, " +COL_NAME+" VARCHAR(50), "+ COL_PRICE+" REAL"+ COL_PRICE+"BLOB)";
        db.execSQL(sql);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        db.execSQL("DROP TABLE IF EXISTS " + TBL_NAME);
        onCreate(db);
    }
    //SELECT
    public Cursor queryData(String sql){
        SQLiteDatabase db = getReadableDatabase();
        return db.rawQuery(sql,null);
    }
    //INSERT, UPDATE, DELETE
    public void execSql(String sql){
        SQLiteDatabase db = getWritableDatabase();
        db.execSQL(sql);
    }
    public int getNumberOfRows(){
        Cursor cursor = queryData("SELECT * FROM "+ TBL_NAME);
        int numberOfRows = cursor.getCount();
        cursor.close();
        return numberOfRows;
    }
    public void createSampleData(){
        if(getNumberOfRows()==0){
            try {
                execSql("INSERT INTO "+ TBL_NAME+" VALUES(null, 'somewhere', 99999)");
                execSql("INSERT INTO "+ TBL_NAME+" VALUES(null, 'Here', 99999)");
                execSql("INSERT INTO "+ TBL_NAME+" VALUES(null, 'There', 99999)");
                execSql("INSERT INTO "+ TBL_NAME+" VALUES(null, 'IDK', 99999)");
                execSql("INSERT INTO "+ TBL_NAME+" VALUES(null, 'For Sure', 99999)");
            }catch (Exception e){
                Log.e("Error: ",e.getMessage().toString());
            }
        }
    }
}

--------------------------------------------endProduct----------------------------------------------  
-------------------------------------------Main-----------------------------------------------------  
package com.PhanVykiet.sqlitetest;

import android.app.AlertDialog;  
import android.app.Dialog;  
import android.content.DialogInterface;  
import android.database.Cursor;  
import android.os.Bundle;  
import android.view.Menu;  
import android.view.MenuItem;  
import android.view.View;  
import android.view.ViewGroup;  
import android.widget.Button;  
import android.widget.EditText;  

import androidx.annotation.NonNull;  
import androidx.appcompat.app.AppCompatActivity;  

import com.PhanVykiet.adapter.TourAdapter;  
import com.PhanVykiet.model.Tour;  
import com.PhanVykiet.sqlitetest.databinding.ActivityMainBinding;  

import java.util.ArrayList;  
import java.util.List;  

public class MainActivity extends AppCompatActivity {  
    ActivityMainBinding binding;  
    TourAdapter adapter;  
    ArrayList<com.PhanVykiet.model.Tour> Tour;  
    Database db;  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        binding = ActivityMainBinding.inflate(getLayoutInflater());  
        setContentView(binding.getRoot());  

        prepareDb();
        loadData();
    }

    private void loadData() {  
        adapter = new TourAdapter (MainActivity.this, R.layout.item_list, getDataFromDb());
        binding.lsvProduct.setAdapter(adapter);
    }

    private List<com.PhanVykiet.model.Tour> getDataFromDb() {  
        Tour = new ArrayList<>();
        Cursor cursor = db.queryData("SELECT * FROM " + Database.TBL_NAME);
        while (cursor.moveToNext()){
            Tour.add(new Tour(cursor.getInt(0), cursor.getString(1), cursor.getDouble(2)));
        }
        cursor.close();
        return Tour;
    }

    public void openEditDialog(com.PhanVykiet.model.Tour p){
        //Log.i("test", "ok");
        Dialog dialog = new Dialog(this);
        dialog.setContentView(R.layout.dialog_edit);

        EditText edtName = dialog.findViewById(R.id.edtName);
        edtName.setText(p.getTourName());
        EditText edtPrice = dialog.findViewById(R.id.edtPrice);
        edtPrice.setText(String.valueOf(p.getTourPrice()));

        Button btnSave = dialog.findViewById(R.id.btnSave);
        btnSave.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String name = edtName.getText().toString();
                Double price = Double.valueOf(edtPrice.getText().toString());
                db.execSql("UPDATE " + Database.TBL_NAME + " SET " + Database.COL_NAME + "='" + name + "', " + Database.COL_PRICE + "=" + price + " WHERE " + Database.COL_CODE + "=" + p.getTourCode());
                loadData();
                dialog.dismiss();
            }
        });

        Button btnCancel = dialog.findViewById(R.id.btnCancel);
        btnCancel.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dialog.dismiss();
            }
        });

        dialog.getWindow().setLayout(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.WRAP_CONTENT);
        dialog.show();
    }

    public void openDeleteConfirmDialog(com.PhanVykiet.model.Tour p){
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("Xac nhan xoa!");
        builder.setIcon(android.R.drawable.ic_delete);
        builder.setMessage("Ban co chac muon xoa sp '" + p.getTourName() + "'?");

        builder.setPositiveButton("Ok", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                db.execSql("DELETE FROM " + Database.TBL_NAME + " WHERE " + Database.COL_CODE + " = " + p.getTourCode());
                loadData();
                dialog.dismiss();
            }
        });

        builder.setNegativeButton("Cancel", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                dialog.dismiss();
            }
        });

        Dialog dialog = builder.create();
        dialog.getWindow().setLayout(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.WRAP_CONTENT);
        dialog.show();
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.option_menu, menu);
        return super.onCreateOptionsMenu(menu);
    }

    @Override
    public boolean onOptionsItemSelected(@NonNull MenuItem item) {
        if (item.getItemId() == R.id.mnAdd){
            Dialog dialog = new Dialog(this);
            dialog.setContentView(R.layout.dialog_add);

            EditText edtName = dialog.findViewById(R.id.edtName);
            EditText edtPrice = dialog.findViewById(R.id.edtPrice);

            Button btnSave = dialog.findViewById(R.id.btnSave);
            btnSave.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    //Insert
                    String name = edtName.getText().toString();
                    Double price = Double.valueOf(edtPrice.getText().toString());
                    db.execSql("INSERT INTO " + Database.TBL_NAME + " VALUES(null, '" + name + "', " + price + ")");
                    loadData();
                    dialog.dismiss();
                }
            });

            Button btnCancel = dialog.findViewById(R.id.btnCancel);
            btnCancel.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    dialog.dismiss();
                }
            });

            dialog.getWindow().setLayout(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.WRAP_CONTENT);
            dialog.show();
        }
        return super.onOptionsItemSelected(item);
    }

    private void prepareDb() {
        db = new Database(MainActivity.this);
        db.createSampleData();
    }
}
-------------------------------------endMain-----------------------------------------------  
------------------------------------activity_main_xml---------------------------------------  
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/tvTitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:textSize="25dp"
        android:gravity="center"
        android:background="@color/black"
        android:textColor="@color/white"
        android:text="Thông tin "/>


    <ListView
        android:id="@+id/lsvProduct"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@id/tvTitle"/>
</RelativeLayou>      
-------------------------------endActivity-Main_xml---------------------------------   
  
-------------------------------item_list_xml-----------------------------------------  
  
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="10dp"
    android:weightSum="10">

    <TextView
        android:id="@+id/txtInfo"
        android:layout_width="0dp"
        android:layout_weight="9"
        android:layout_height="35dp"
        android:textSize="26sp"
        android:textStyle="bold"
        android:text="Product Info"/>

</LinearLayout>  
-----------------------------enditem_list_xml------------------------------------------------------  


-----------------------------dialog_add_xml------------------------------------------------------  
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:padding="10dp"
    android:layout_height="match_parent">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="ADD"
        android:gravity="center"
        android:textSize="40sp"
        android:textStyle="bold"/>

    <EditText
        android:id="@+id/edtName"
        android:layout_marginTop="10dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Tour name"
        android:textSize="26sp"/>

    <EditText
        android:id="@+id/edtPrice"
        android:layout_marginTop="10dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Tour price"
        android:textSize="26sp"/>

    <LinearLayout
        android:layout_width="match_parent"
        android:orientation="horizontal"
        android:layout_marginTop="10dp"
        android:layout_height="wrap_content"
        android:gravity="end">

        <Button
            android:id="@+id/btnSave"
            android:text="Save"
            android:backgroundTint="#00D12D"
            android:textSize="22sp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

        <Button
            android:id="@+id/btnCancel"
            android:text="Cancel"
            android:layout_marginStart="10dp"
            android:backgroundTint="#1304E1"
            android:textSize="22sp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

    </LinearLayout>

</LinearLayout>
-----------------------------ENDdialog_add_xml------------------------------------------------------  


-----------------------------dialog_edit_xml------------------------------------------------------  
  <?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:padding="10dp"
    android:layout_height="match_parent">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="EDIT"
        android:gravity="center"
        android:textSize="40sp"
        android:textStyle="bold"/>

    <EditText
        android:id="@+id/edtName"
        android:layout_marginTop="10dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Tour name"
        android:textSize="26sp"/>

    <EditText
        android:id="@+id/edtPrice"
        android:layout_marginTop="10dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Tour price"
        android:textSize="26sp"/>

    <LinearLayout
        android:layout_width="match_parent"
        android:orientation="horizontal"
        android:layout_marginTop="10dp"
        android:layout_height="wrap_content"
        android:gravity="end">

        <Button
            android:id="@+id/btnSave"
            android:text="Save"
            android:backgroundTint="#00D12D"
            android:textSize="22sp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

        <Button
            android:id="@+id/btnCancel"
            android:text="Cancel"
            android:layout_marginStart="10dp"
            android:backgroundTint="#1304E1"
            android:textSize="22sp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

    </LinearLayout>

</LinearLayout>


-----------------------------Enddialog_edit_xml------------------------------------------------------  
