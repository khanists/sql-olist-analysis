# Kategori Produk dengan Kontribusi Revenue Terbesar

Analisis SQL pada dataset Brazilian E-Commerce (Olist) menggunakan BigQuery.

## Pertanyaan Bisnis

Kategori produk apa yang paling besar kontribusinya terhadap revenue, dan apakah revenue terkonsentrasi di beberapa kategori saja?

## Query

Revenue per kategori (nama kategori diterjemahkan ke bahasa Inggris):

```sql
SELECT t.product_category_name_english AS category,
       SUM(oi.price) AS revenue
FROM `belajar-sql-512905.olist.order_items` oi
JOIN `belajar-sql-512905.olist.products` p
  ON oi.product_id = p.product_id
LEFT JOIN `belajar-sql-512905.olist.category_translation` t
  ON p.product_category_name = t.product_category_name
GROUP BY category
ORDER BY revenue DESC;
```

Total revenue keseluruhan (sebagai penyebut persentase):

```sql
SELECT SUM(price) AS total_revenue
FROM `belajar-sql-512905.olist.order_items`;
```

## Temuan

- Kategori teratas adalah **health_beauty** dengan revenue sekitar 1,26 juta BRL, atau **9,3%** dari total revenue 13,59 juta BRL. Disusul watches_gifts dan bed_bath_table.
- 3 kategori teratas menyumbang **25,8%** dari total revenue, dan 10 kategori teratas menyumbang **62,4%**.
- Olist memiliki sekitar 72 kategori, sehingga revenue **cukup terkonsentrasi**: sekitar 14% kategori menghasilkan lebih dari 60% revenue.

10 kategori teratas: health_beauty, watches_gifts, bed_bath_table, sports_leisure, computers_accessories, furniture_decor, cool_stuff, housewares, auto, garden_tools.

## Catatan

- Angka mencakup semua item di tabel `order_items` tanpa memfilter status order.
- Sebanyak 1.627 item (sekitar 186 ribu BRL, atau 1,4% dari total revenue) tidak memiliki kategori atau terjemahan kategori, sehingga tidak masuk ke daftar kategori di atas. Persentase konsentrasi tetap dihitung terhadap total revenue semua item.
- Nilai `price` dalam Real Brasil (BRL).
