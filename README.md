# android-search-in-recyclerview

## AndroidManifest.xml
```xml
<uses-permission android:name="android.permission.INTERNET" />
```

## build.gradle (app)
```
implementation 'com.squareup.retrofit2:retrofit:2.9.0'
implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
```

## Beneficiary.kt
```java
data class Beneficiary(val alias: String, val accountNumber: String)
```

## IntenseTundraService.kt
```java
package com.example.recyclerviewcallapi

import com.google.gson.JsonObject
import retrofit2.Call
import retrofit2.http.GET

interface IntenseTundraService {
    @GET("beneficiaries")
    fun getBeneficiaries(): Call<JsonObject>
}
```

## SimpleAdapter.kt
```java
package com.example.recyclerviewcallapi

import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.Filter
import android.widget.Filterable
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView
import com.example.recyclerviewcallapi.SimpleAdapter.*
import java.util.*

class SimpleAdapter(
    private var listItems: List<Beneficiary>
) : RecyclerView.Adapter<SimpleViewHolder>(), Filterable {

    val listItemsFull = listItems

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): SimpleViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.list_item, parent, false)
        return SimpleViewHolder(view)
    }

    override fun onBindViewHolder(holder: SimpleViewHolder, position: Int) {
        val beneficiary = listItems[position]
        holder.bind(beneficiary)
    }

    override fun getItemCount(): Int {
        return listItems.size
    }

    class SimpleViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        fun bind(beneficiary: Beneficiary) {
            itemView.findViewById<TextView>(R.id.textView1).text = beneficiary.alias
            itemView.findViewById<TextView>(R.id.textView2).text = beneficiary.accountNumber
        }
    }

    override fun getFilter(): Filter {
        return simpleFilter
    }

    private val simpleFilter = object : Filter() {

        override fun performFiltering(searchTerm: CharSequence?): FilterResults {
            val filteredDataSet = if (searchTerm == null || searchTerm.isEmpty()) {
                listItemsFull
            } else {
                listItemsFull.filter {
                    it.alias.toLowerCase(Locale.getDefault())
                        .startsWith(searchTerm.toString().toLowerCase(Locale.getDefault()).trim())
                }
            }
            val results = FilterResults()
            results.values = filteredDataSet
            return results
        }

        override fun publishResults(p0: CharSequence?, results: FilterResults) {
            listItems = results.values as List<Beneficiary>
            notifyDataSetChanged()
        }
    }

}
```

## MainActivity.kt

```java
package com.example.recyclerviewcallapi

import android.os.Bundle
import android.view.Menu
import android.widget.SearchView
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.google.gson.JsonObject
import retrofit2.Call
import retrofit2.Callback
import retrofit2.Response
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

class MainActivity : AppCompatActivity() {

    private lateinit var simpleAdapter: SimpleAdapter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val recyclerView = findViewById<RecyclerView>(R.id.recyclerView)
        recyclerView.layoutManager = LinearLayoutManager(this)

        val retrofit = Retrofit.Builder()
            .baseUrl("https://the-name-12345.herokuapp.com")
            .addConverterFactory(GsonConverterFactory.create())
            .build()

        val service = retrofit.create(IntenseTundraService::class.java)

        service.getBeneficiaries().enqueue(object : Callback<JsonObject> {
            override fun onResponse(call: Call<JsonObject>, response: Response<JsonObject>) {
                val body = response.body()
                val beneficiaries = parse(body)
                simpleAdapter = SimpleAdapter(beneficiaries)
                recyclerView.adapter = simpleAdapter
            }

            override fun onFailure(call: Call<JsonObject>, t: Throwable) {
                TODO("Not yet implemented")
            }
        })
    }

    private fun parse(body: JsonObject?): List<Beneficiary> {
        if (body == null) return emptyList()
        return body["value"].asJsonObject["beneficiaries"].asJsonArray.map {
            Beneficiary(
                it.asJsonObject["alias"].asString,
                it.asJsonObject["accountNumber"].asString
            )
        }
    }

    override fun onCreateOptionsMenu(menu: Menu): Boolean {
        menuInflater.inflate(R.menu.menu_search, menu)
        val menuItem = menu.findItem(R.id.menu_item_search)
        val searchView = menuItem.actionView as SearchView
        searchView.setOnQueryTextListener(object : SearchView.OnQueryTextListener {
            override fun onQueryTextSubmit(p0: String?): Boolean {
                return false
            }

            override fun onQueryTextChange(term: String?): Boolean {
                simpleAdapter.filter.filter(term)
                return true
            }
        })
        return super.onCreateOptionsMenu(menu)
    }

}
```

## res/layout/list_item.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:paddingHorizontal="16dp"
    android:paddingVertical="8dp">

    <TextView
        android:id="@+id/textView1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <TextView
        android:id="@+id/textView2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

</LinearLayout>
```

## res/menu/menu_search.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/menu_item_search"
        android:title="Search"
        app:actionViewClass="android.widget.SearchView"
        app:showAsAction="always" />

</menu>
```

## activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```
