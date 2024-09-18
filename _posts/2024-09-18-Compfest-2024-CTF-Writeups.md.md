---
title: Compfest 2024 CTF Writeups (Bahasa Indonesia)
author: Naufal Ardhani
date: 2024-09-18 11:33:00 +0800
categories:
  - CTF
  - CTF-Writeups
tags:
  - SQL-Injection
  - Prototype-Pollution
  - Mass-Asignment
---
## Table of Contents
- [Chicken Daddy (375 pts) - SQL Injection Lead to File Read](#chicken-daddy-375-pts)
	- SQL Injection - Union Based
	- `secure_file_priv` disabled
	- Read File with `load_file('filename.txt')`
- [Siak-OG (408 pts)](#siak-og-408-pts)
	- Prototype Pollution
- [Copasbin (490pts)](#copasbin-490pts)
	- Blind XSS (Cross Site Scripting)
	- Bypass sanitize() with allowedKey - `express-xss-sanitizer`
	- Mass Asignment - `lodash.merge()`
	- Prototype Pollution

## Chicken Daddy (375 pts)
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXe3PPGvJgG_oXKIBn0VujOu3-ulwBP7S5oNpCSimxNlPoYvc-KR-JTzzLqZ6dMK_DwqZHZKA5P8KwzEMgovh8oYyb00V3Rrxc8D8WwbPBf-cZ8xzVqhbDyAKqrdLDTuF0HcAjwakB71EXJ121O7B5g2UuhN?key=VX-8OidzEmhvroEY3hI9tQ)

Diberikan sebuah url dan source code dari challenge, terdapat hal menarik saat melakukan code review, ketika parameter `id` available maka si web challenge akan memanggil fungsi getRecipe(id).

**![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdl11Ivt3_R9O-n_hnDCSsD-Cx99H70VPuFYRr_7B0yZVfoJLz6vVahPkjuu0QyDdCFwNOBoSsKzJDxp8kbnE-rKZ2g5dKrOqLeT-jISoEol1ny0V7-R3ULrjJL-s2d32r_eesyoE0_UmKvBb_RNmwveyI9?key=VX-8OidzEmhvroEY3hI9tQ)**

**

Untuk mengetahui apa isi dari fungsi getRecipe() saya melakukan pengecekan darimana getRecipe() berasal, ternyata fungsi getRecipe() di-impor dari file ./database.js.

  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXczbzTK-VSLmjo4v5Ja60pvJXW-rtCUAED0Sh41kVBOEd62duhbWBdqiwI0GYJ0LWSUVkgCjDbGEZ9bojY80piOBrVLlwnLMuKZR4CdQNXDEeHOQWyv2rNFNp53RtiNymf8ZzBUm1-QT28M379R4bDLi24?key=VX-8OidzEmhvroEY3hI9tQ)

  

Berikut ini isi dari fungsi getRecipe() di ./database.js:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXc6i4TJ7n0LxDCe2K_mpZGJJROu1QEpHzhmsKycM8Z6dGfL67t_GXd9tHS8mKOVRAaxFx7Nmtt22KC-bFhzKmxxhNSbZUrOUlhQ3_d8QZFRA2eeyy2PsnJhWUrkok0ao3MyijoVwazIRkJ8SAfgHQUAhKBv?key=VX-8OidzEmhvroEY3hI9tQ)

  

Ternyata isi fungsi getRecipe(id) untuk koneksi database untuk melakukan query mengambil data dari table recipes, letak celahnya ada pada ${id}, itu menyebabkan sql injection karena tidak adanya sanitasi terhadap input user, jadi attacker bisa melakukan perintah sql dengan menginputnya di parameter `id`.

  

Saya menggunakan SQL Injection union based untuk melakukan eksploitasi, dan mendapatkan info tambahan untuk jumlah kolom pada tale recipes.

  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXc9YpnS8AWCxLisqjfSuYF1tXHqDhnEK_pIlCH0htHVYwScTPlsjPzmkDs1fBERDNUlCeX5nZe2ysyiQpfwz_v3XRVc6yoTu5wk0FoITlM-AfGUOER25HJVYMuMZREZYA2yMVizCAUGq977-NiJ9E-RbhA?key=VX-8OidzEmhvroEY3hI9tQ)

  

Jadi untuk melakukan union based, saya hanya tinggal menggunakan query `+union+select+(banyak_kolom)--.

Dengan teknik union-based saya mendapati user root.

  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdVF-uybG0ahz7yIyYXxq2L3o2rDHfGuX53pDKTJQiaPCZdiZBSGpS-9Kb3qO9SnjPxVMMoqkN_ni9LS7ST_VnJ_2UqjDkAbcASEZwF_CM-GNJV0p5uK_M7iev1U2lPlb_imx1eWaiPYnPRN4PxWnsUYgy7?key=VX-8OidzEmhvroEY3hI9tQ)

  

Pada file /etc/mysql/my.cnf terdapat `secure-file-priv` yang kosong/tidak diisi, Jika ini kosong, MySQL tidak membatasi lokasi file yang dapat diakses. Mengatur direktori yang tepat pada opsi ini dapat meningkatkan keamanan dengan membatasi lokasi file yang dapat diakses oleh MySQL.

  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfT15Qz1kVPjqxhzdTt6w6cwYuWnCKW9Y-bkJhF7gwvWgBM299E7nU72MT-yMmzvGxRU1WJxXLOCSeLo8zqevEmTmiMvKDZE0URCHqjf_NZVp2xBBMRhTPbOr8Y3FVa2Kmx8jd9KVqEJAwNd2mppVi_dG0t?key=VX-8OidzEmhvroEY3hI9tQ)

  

Artinya attacker bisa melakukan load_file(file_path) untuk membaca file pada localhost service, saya langsung mencari cara untuk mendapatkan flag. 

  

pada Dockerfile root directory si user yang menyimpan flag.txt dirahasiakan dengan `redacted`.

  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfVZF5-4MMN_TGJYDFzHNTuKOmciorKCBIr7VkEjbAFjx8CUM3jgYq4UXG4hAXi-kdU3-84YGBoRx8yh65aXPdzZJlmVzYFN8BlZ1owPj45UZunBA9w_ippBLRTgSugGkebcWVW2bYIEqu5tG99Rz5M5l7B?key=VX-8OidzEmhvroEY3hI9tQ)

  

Untuk mengetahui user itu kita bisa menggunakan file /etc/passwd yang berisi daftar semua akun pengguna yang terdaftar di sistem. Setiap baris dalam file ini mewakili satu pengguna dan berisi berbagai informasi terpisah oleh tanda titik koma (:).

  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdfZUZ2_Sg7Kg4ebMXHNoLmtYFNOSLbV4dcGgJR9yPtiHiniXsrkoz4TOp-zn9VtYJdUmZJF46ADnR7A9Ilpw_Xuj_SVzT_Zs47z6XZ-0i0K0k1pPSPTlsY91q-VoG5FkkDACXhpkJBmnd9kziDQ21xxJft?key=VX-8OidzEmhvroEY3hI9tQ)

  

Dari file /etc/passwd saya mendapatkan informasi nama user, dan hanya tersedia 1 user yaitu `ayamCemani`, untuk mendapatkan flag kita bisa mengambilnya dengan load_file(‘/home/ayamCemani/flag.txt’)

  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdDAVkn7LC73k1NveqzzVmIOerOR3gfI4h2SuL569PJOHNtlRDDA6ItnRvecX1qUyYZlpBK--2dGR9R9YmlgIT6rP90cq5nlV_CccQL4Hmx9zaOiWembY1KUGEcW57exkFgAbSeVZswgQc7dpH0053o-tA?key=VX-8OidzEmhvroEY3hI9tQ)

  
> Di bawah ini merupakan scripting untuk menyelesaikan soal.
{: .prompt-info }

```python
import re
import requests

URL = "http://challenges.ctf.compfest.id:9014/?id=-2+union+select+1,load_file(%27file_path%27),3,4,5--"

def parsing_res(text):
	print( re.findall('<h1 class="text-3xl font-bold">(.*?)</h1>', text))

def load_file(file_path):
	r = requests.get(URL.replace("file_path", file_path))
	return r.text

if __name__ == "__main__":
	flag = load_file('/home/ayamCemani/flag.txt')
	parsing_res(flag)
```
Flag: COMPFEST16{d0_Not_d1Sabl3_@@sECur3_f1l3_pr1V!!!_5a91f7c870}

## Siak-OG (408 pts)
Diberikan sebuah netcat service yang berguna untuk mendapatkan url service dan juga ada source codenya.
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfcvcuLL8srwzj5GF-6eowc_EKup94q9WbCWi9xPTVPxmxlt31c7Mdpy1msp_FpQ8I64ATP2kblIbGlqD233G8swqtdOQ_GMF13qQlrq9_hC1Aigv3bxq0Uix5NmwuKfPUE81z7vCHrgY3p3mn7CTj99Yc_?key=VX-8OidzEmhvroEY3hI9tQ)
Di awal code review saya menemukan apa yang harus dilakukan, yaitu membaca flag melalui courses_list[‘DSA’].
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcBkfPqPPjDxy--T9nt-Qcb6xggciK58s0yoc4O26IsvRMKnF_k3r8lkT1clpFMRYYcObZd14DaOnohcpXHCb2_U0OLLo4svGNpY0VwTROvifuBj_yqhhaENTx0E5L0gDJAeZcT-ARUCKBowbGE_YLFmWI?key=VX-8OidzEmhvroEY3hI9tQ)
Terdapat api untuk edit irs, mungkin saya bisa mengambil irs DSA tadi agar dapat flag.
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXd7a9jHfc_Vj2zXMdwW5D9ue7Ev3EVk9CiefP4mkVxK1C7uUlptd07Y1HYp6BesGGD_ePy7y7kuGnYu2sLyd2rY67bEV4do7Sibo_PPg79lLnc5nusqVLfC6dgaTXO_Qbt6pu9V42AeuZjBl1SgDNNV-GA9?key=VX-8OidzEmhvroEY3hI9tQ)
Coba menggunakan repeater burp untuk mengambil course DSA seperti berikut.
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXf9N0MqFBvr_ZZodtCSjfa8B6UxLqwRb8aBIzDKw3rZ9OzGZjXxt01VF6AKr0UkVzOiUcKiRg8P4T5NML_V_UZB6zqIv-zNA5CH2-4i-jRfEG2YTpQtlHWfTe__xmM8-SXnylCBdZyhho0aiXXga7JyqEg?key=VX-8OidzEmhvroEY3hI9tQ)
Tapi itu tidak berhasil.
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeyQUPjVMjeTc28XZMJcn6h12Hk6uNdtDh4c7DhS0apJHPl0X87oQj13X0tslkhug4WX6wIazfgeqjcqBwm5J6g0kwIlDgSVg089xthuzcMg5TcWIr4gsAZKXj4wJIt-waPmAa7hZNajnWwT-5DjxVGBMKY?key=VX-8OidzEmhvroEY3hI9tQ)
Saat melakukan code review ulang dan debugging saya menyadari hanya ada logika di bawah, yang artinya tidak ada sama sekali pernyataan untuk meng-set req.session.admin = false jika ip bukan dari 127.0.0.1.
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfeJttVygqgCJ9Id9dnJAGUCEdBuc12zIGzORr0QW4wrOf0gJrBQr39e3JViVA5m5m4obghq1L_ARSvWEXaOHloneSpPuc88RpltzYUPF5v5yxj0iN3AqH5DMSc807STnB7M__L8rUDK4KWk9kJeZxpFc4?key=VX-8OidzEmhvroEY3hI9tQ)
Dan juga mendapati pada /api/v1/edit-irs vulnerable terhadap prototype pollution, yang artinya attacker dapat meng-set object admin agar bisa mengambil course yang tidak available seperti course DSA.
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeGCVgPdwJrBc2638BeT7fv-GhIKx6nNeEeLWsYShxaNEn7po_T08HJyj-p5otBcJeVlcnoesTkGVm0rrbOc5n86ZHBVJvFNrryQnGcV9CzZKT7sNYG8a6OTbt8d3lYeUvIFY6Ko4GOLn7VrV1Efp1kCuEV?key=VX-8OidzEmhvroEY3hI9tQ)
Berikut http request untuk meng-set object admin dan mengambil course DSA.
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfUmgbus7EMxrrXGWpJhmDiO7eeK3V6cIJZLF3dgUTCcYY2_HyWd-9cC359kplfhTrhTu50nt2BIoLW_utRFKzZn_x-otBP8eDPsWvEOxncsv3sFaNGTAIdUXWmIsNONh6QDm00hGC5igcfHdm6SbfoP9cc?key=VX-8OidzEmhvroEY3hI9tQ)
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXemKLcrfejdHoLjyfph-ohvJUQBDIFeKXyS7YK0xnLsubwq-EMlBpAMOmGfRym4WztusD80gXKFfBE29H7RIBcT8_lBkxQNCh3eclhJ98iI0MU3H_qdx7weWtsTuH9tmW1CwJy253Q6mBK_LGy04OLsIp4R?key=VX-8OidzEmhvroEY3hI9tQ)
Flag: COMPFEST16{n0w_c4n_y0u_h3lp_me_w1th_th1s_1rl?_2857a76eba}

## Copasbin (490pts)
Terdapat 2 lib yang digunakan di express service
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcWbcCHVugsd41YfqOQvHrikO_ZM8LjLBTskZJnWYaZqbEmHfKzSc0SD_MdH7TuD6t2AzSUVt3RzbWG1tF0aktwslrVeW_5PLkLw_2HOgKacbF7Ps9QzdnOWyoNC2IKOYyMoLawzkyOXdf-1AgXqFMQRqEA?key=VX-8OidzEmhvroEY3hI9tQ)
Juga terdapat vulnerable prototype pollution di fungsi SanitizerObject()
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfpy9kru6gI6N9f3dgSW4WmOUoQ-0d6Ikpu0pKQacOCfFQ3SGlacUaHk_G_HvEnkbzWYKpQVdPUbpHApH-XbMvZSWMhglXeD_CSbSTH2DpQ4pB7RIcG6i_h8XAy31jief1gniV5VhwKaFcuP_J_PO9623dU?key=VX-8OidzEmhvroEY3hI9tQ)
Ada path traversal juga karena input filepatch tidak disanitasi, bodohnya saya ini ngga dicoba karna sok sok an mau intended dari probset dan ya, ampe kelar kelupaan buat eksploit ini.
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeU4OPznTxndjwm3lHCHd7gim6SmoaEpjMYVniEzmhK-emT512lPQmi8ZiN4R6iUCX3iXMB0VloflG43V_8T3FvUx3woUbUcW8qkYaVRGJwMr1d_cWy37DbgKyRoIJZ-ma4ib5chTw6GYuz6VzbBKmrQOd1?key=VX-8OidzEmhvroEY3hI9tQ)
Diketahui dari admin bahwa intended dari challenge ini adalah XSS, pada [POST] /api/copas juga vulnerable mass asignment lodash.merge().
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfFmUv-I8H98V5pate_u0LOppeptmcNSkEzmsm-nlmd0OmSempoXi_mogM9gL3GTUnIkLIjN6PN2JRn2g04oTr2DuW-l05EcPOzQoPeTHbdL7VXUVPWzmxnN3FVkYznnNrFKMsGVrZc2M4WJCHNNewy41SK?key=VX-8OidzEmhvroEY3hI9tQ)
Attacker bisa melakukan 2 cara untuk membuat payload xss, keduanya sama sama memanfaatkan cara yang sama untuk bypass fungsi sanitize.  

Cara 1 (Prototype Pollution):
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeXLueIYOl9OHRuaivhWZDKCOPwi1wkENgfzdEu2-kgiThhG-BdZvhteyE1SyV7c0abrpcFK5JUfq96k8oWLSwpBC823fzcpRiXDTBuPobkQUsKI3J5ejZ3_DXcNasE0wSWi_b500g8eoj0G2YhzlAOZYA?key=VX-8OidzEmhvroEY3hI9tQ)


Cara 2 (Mass Assignment):
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeJw1VuQ21Q9LjrpQvh7YIoIQzJdgEg7zG83N-UrUyCAJusn3Msoqx1dENL8HDKIy7KdQbzaWwvF9iI_-bioubPP59Ry4e22biqV3MHIgiuTnnTy0jPvLAwsGIIj8WWTITnAX5tWcQw7vO_aDFY87wzPKWE?key=VX-8OidzEmhvroEY3hI9tQ)

Saya menggunakan mass asignment dan payload xss dengan tag <img>.
![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfQrPiFw5AtXlsUWiTNMxpWaEg99HLAXx9XZt7T6wxIKAB1FqYgEXMKrqQZ4A7ENVFJ6QF7sLUZ3g7NA8YTDDvTzcdPZ7A0CNRsxq7_SEt7A9ZqZ8pd-Ax5g_q23RCusOtRqmvpi7RfAqAiqK7DS1EA36BF?key=VX-8OidzEmhvroEY3hI9tQ)

  ![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcF7EtPvmTgmj_zWXjUHOJws_OFUvFO3UwUKX2eoRzDMOvZUkNYZV7vLomcIywB1cOT5sDhJA0XDjNORXgA_zULyMzj9bXnpCveNE69a97UKkT5zYe9W_FB8qhsL6ttOX4BRsIG7g7dmC24V7MhSHm4wVNh?key=VX-8OidzEmhvroEY3hI9tQ)
Flag didapatkan dan ternyata ini merupakah sebuah cve, karena dari awal pengerjaan saya hanya melakukan code review, debugging, dan testing langsung di api.
Flag: COMPFEST16{L0Ve_m3_s0me_W3B_CVEs_309f7ecc3d}