# Order yang Gagal Menjadi Penjualan

Analisis SQL pada dataset Brazilian E-Commerce (Olist) menggunakan BigQuery.

## Pertanyaan Bisnis

Berapa persen order yang gagal menjadi penjualan, dan apa penyebab utamanya?

Definisi: order gagal adalah order di tabel `orders` yang tidak memiliki item sama sekali di tabel `order_items`.

## Query

Persentase order gagal terhadap total order:

```sql
WITH failed AS (
  SELECT COUNT(*) AS failed_orders
  FROM `olist.orders` AS o
  LEFT JOIN `olist.order_items` AS oi
    ON o.order_id = oi.order_id
  WHERE oi.order_id IS NULL
),
total AS (
  SELECT COUNT(*) AS total_order
  FROM `olist.orders`
)
SELECT f.failed_orders,
       t.total_order,
       ROUND(100 * f.failed_orders / t.total_order, 2) AS failed_pct
FROM failed f
CROSS JOIN total t;
```

Penyebab utama (order gagal per status):

```sql
WITH failed_by_status AS (
  SELECT o.order_status, COUNT(*) AS failed_orders
  FROM `olist.orders` AS o
  LEFT JOIN `olist.order_items` AS oi
    ON o.order_id = oi.order_id
  WHERE oi.order_id IS NULL
  GROUP BY o.order_status
),
total_failed AS (
  SELECT SUM(failed_orders) AS total
  FROM failed_by_status
)
SELECT fs.order_status,
       fs.failed_orders,
       ROUND(100 * fs.failed_orders / tf.total, 1) AS pct_of_failed
FROM failed_by_status fs
CROSS JOIN total_failed tf
ORDER BY fs.failed_orders DESC;
```

## Temuan

- Dari 99.441 order, 775 order (0,78%) tidak memiliki item sehingga tidak menjadi penjualan.
- Status dominan adalah **unavailable** (77,8% dari order gagal), disusul **canceled** (21,2%). Sisanya (sekitar 1%) berstatus lain.
- Persentasenya kecil, sehingga dampaknya terhadap penjualan keseluruhan terbatas. Namun hampir 8 dari 10 order gagal disebabkan barang tidak tersedia, yang menunjuk ke masalah ketersediaan stok, bukan keputusan pelanggan.

## Catatan

- Order gagal didefinisikan sebagai order tanpa baris di `order_items`. Order yang berstatus `canceled` tetapi masih punya item tidak dihitung di sini.
- Ditemukan 8 order berstatus `delivered` tetapi tanggal pengirimannya kosong (`order_delivered_customer_date` NULL), yaitu anomali pencatatan data. Jumlahnya sangat kecil dibanding total order, dan order ini tidak memengaruhi definisi order gagal sehingga tetap dipertahankan dalam analisis.
