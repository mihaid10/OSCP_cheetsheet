# WebApplication

### robots.txt / sitemap.xml

ウェブアプリケーションは、検索エンジンのボットがサイトをクロールしてインデックスを作成するのを助けるために、サイトマップファイルを含めることができる

これらのファイルには、どのURLをクロールしないかというディレクティブも含まれている

```bash
curl https://www.google.com/robots.txt

---
User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
Disallow: /sdch
Disallow: /groups
Disallow: /index.html?
Disallow: /?
Allow: /?hl=
Disallow: /?hl=*&
Allow: /?hl=*&gws_rd=ssl$
Disallow: /?hl=*&*&gws_rd=ssl
Allow: /?gws_rd=ssl$
Allow: /?pt1=true$
Disallow: /imgres
Disallow: /u/
```



### LFI(Local File Inclution)

シェルペイロードをローカルに書き込みそれをインクルードさせることでリバースシェルを取得する手法。主にPHPアプリが脆弱性の対象となる。



### hosts

hostsを設定しないとアプリがうまく動かない場合も多いのでhosts設定できる場合する

```bash
 sudo vi /etc/hosts
 
 -------
 172.50.3.126    management_portal
```

