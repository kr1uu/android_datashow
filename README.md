# android_datashow
---------------------Adapter----------------------------------  
import android.content.Context;  
import android.view.LayoutInflater;  
import android.view.View;  
import android.view.ViewGroup;  
import android.widget.BaseAdapter;  
import android.widget.TextView;  

import com.myapplication.de2.MainActivity;  
import com.myapplication.de2.Product;  
import com.myapplication.de2.R;  

import java.util.List;

public class ProductAdapter extends BaseAdapter {  
    MainActivity context;  
    private int layout;  
    private List<Product> productList;  
    private LayoutInflater inflater;  

    public ProductAdapter(Context context, int layout, List<Product> productList) {
        this.context = (MainActivity) context;
        this.layout = layout;
        this.productList = productList;
        this.inflater = LayoutInflater.from(context);
    }

    @Override
    public int getCount() {
        return productList.size();
    }
    @Override
    public Object getItem(int position) {
        return productList.get(position);
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
            convertView = inflater.inflate(layout, null);

            holder.txtInfo = convertView.findViewById(R.id.txtInfo);
            convertView.setTag(holder);
        } else {
            holder = (ViewHolder) convertView.getTag();
        }
        Product p = productList.get(position);
        holder.txtInfo.setText(p.getPhoneName()+" - "+String.format("%.0fd", p.getPrice()));

        return convertView;
    }

    static class ViewHolder {
        TextView txtInfo;
    }
}  
----------------------------endAdapter-------------------------------------  
----------------------------Database---------------------------------------  
package com.myapplication.de2;

import android.content.Context;  
import android.database.Cursor;  
import android.database.sqlite.SQLiteDatabase;  
import android.database.sqlite.SQLiteOpenHelper;  
import android.util.Log;  

import androidx.annotation.Nullable;  

public class Database extends SQLiteOpenHelper {  
    public static final String DB_NAME = "Phone.sqlite";  
    public static final int DB_VERSION = 1;  
    public static final String TBL_NAME = "Product";  
    public static final String COL_CODE = "PhoneCode";  
    public static final String COL_NAME = "PhoneName";  
    public static final String COL_PRICE = "Price";  

    public Database(@Nullable Context context) {
        super(context, DB_NAME, null, DB_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        String sql = "CREATE TABLE IF NOT EXISTS " + TBL_NAME + " (" + COL_CODE+" INTEGER PRIMARY KEY AUTOINCREMENT, " +COL_NAME+" VARCHAR(50), "+ COL_PRICE+" REAL)";
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
                execSql("INSERT INTO "+ TBL_NAME+" VALUES(null, 'Vertu constellation', 99999)");
                execSql("INSERT INTO "+ TBL_NAME+" VALUES(null, 'Iphonr5S', 99999)");
                execSql("INSERT INTO "+ TBL_NAME+" VALUES(null, 'NokiaLumia925', 99999)");
                execSql("INSERT INTO "+ TBL_NAME+" VALUES(null, 'SamsungGalaxyS4', 99999)");
                execSql("INSERT INTO "+ TBL_NAME+" VALUES(null, 'HTCDesire500', 99999)");
                execSql("INSERT INTO "+ TBL_NAME+" VALUES(null, 'HKPhoneRevoLEAD', 99999)");
            }catch (Exception e){
                Log.e("Error: ",e.getMessage().toString());
            }
        }
    }
}  
---------------------------------------endDatabase----------------------------------  
---------------------------------------ClassProduct----------------------------------  
package com.myapplication.de2;

public class Product {
    int PhoneCode;  
    String PhoneName;  
    double Price;  

    public Product(int PhoneCode, String PhoneName, double Price) {
        this.PhoneCode = PhoneCode;
        this.PhoneName = PhoneName;
        this.Price = Price;
    }

    public int getPhoneCode() { return PhoneCode; }

    public void setPhoneCode(int phoneCode) {
        this.PhoneCode = PhoneCode;
    }

    public String getPhoneName() {
        return PhoneName;
    }

    public void setPhoneName(String PhoneName) {
        this.PhoneName = PhoneName;
    }

    public double getPrice() {
        return Price;
    }

    public void setPrice(double Price) {
        this.Price = Price;
    }

    public String getInfo() {
        return "Phone Code: " + PhoneCode + ", Phone Name: " + PhoneName + ", Price: " + Price;
    }
}  
--------------------------------------------endProduct----------------------------------------------  
-------------------------------------------Main-----------------------------------------------------  
package com.myapplication.de2;  
import android.database.Cursor;  
import android.os.Bundle;  
import android.view.LayoutInflater;  
import android.view.View;  
import android.view.ViewGroup;  
import androidx.appcompat.app.AppCompatActivity;  
import com.myapplication.Adapter.ProductAdapter;  
import com.myapplication.de2.databinding.ActivityMainBinding;  
import java.util.ArrayList;  
import java.util.List;  
import androidx.annotation.NonNull;  
import androidx.appcompat.app.AppCompatActivity;  

public class MainActivity extends AppCompatActivity {  

    ActivityMainBinding binding;    
    ProductAdapter adapter;    
    ArrayList<com.myapplication.de2.Product> productList;    
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
        adapter = new ProductAdapter(MainActivity.this, R.layout.layout, getDataFromDb());
        binding.lsvProduct.setAdapter(adapter);
    }

    private List<com.myapplication.de2.Product> getDataFromDb() {
        productList = new ArrayList<>();
        Cursor cursor = db.queryData("SELECT * FROM " + Database.TBL_NAME);
        while (cursor.moveToNext()){
            productList.add(new Product(cursor.getInt(0), cursor.getString(1), cursor.getDouble(2)));
        }
        cursor.close();
        return productList;
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
  
-------------------------------layout_xml-----------------------------------------  
  
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
-----------------------------endLayout_xml------------------------------------------------------


