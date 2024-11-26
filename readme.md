# DOKUMENTASI

Dokumentasi ini dibuat dengan menggunakan `mkdocs` dengan thema `material` dan plugin `exporter`

Jika anda tidak ingin memasang `mkdocs` anda tetap dapat melihat dokumentasi ini dengan menggunakan `markdown viewer` lainnya dan melihat/mengubah isi dokumentasi di folder `docs`

## Clone Dokumentasi

```sh
```sh
git clone git@github.com:pirsani/docpanda.git
cd docpanda
```

jika ingin mengubah dokumentasi, buat branch baru

```sh
git checkout -b namabranch
```

setelah selesai membuat perubahan simpan dan push

```sh
git add .
git commit -am "keterangan commit"
git push -u origin namabranch
```

## instalasi mkdocs untuk mem

```sh
pip install mkdocs
pip install mkdocs-material
pip install mkdocs-exporter
playwright install 
```

## menjalankan mkdocs

```sh
mkdocs serve
```

## build dokumentasi status

```sh
mkdocs build
```

akan tergenerate folder `site` yang berisi site statis dokumentasi

- referensi

  - <https://www.mkdocs.org/>
  - <https://squidfunk.github.io/mkdocs-material/getting-started/>
  - <https://adrienbrignon.github.io/mkdocs-exporter/getting-started/#installation>