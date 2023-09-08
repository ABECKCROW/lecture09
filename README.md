# MyBatisでCRUDの実装

### ■概要
* GET・・・2種類実装  
①bandsテーブルよりact_Announcement_Dateを除いたidとband_nameをReadする。  
②act_Announcement_Dateが今日以前の①をReadする。

* Create・・・バンド名とアナウンス日を作成。idは自動作成
  
* Patch・・・バンド名とアナウンス日を更新。

* Delete・・・バンド名とアナウンス日を削除。

### ■想定
フェスイベントで出演バンドを順番にアナウンスする。  
アナウンスする日が決まっていてリクエストした今日時点でアナウンス済み(アナウンス日の設定が今日以前)のバンドをレスポンスする。  
また重複するバンド名を省いて一意のデータとしてレスポンスする。
ここまでは実装済み。  
  

### ■DB登録のテーブル

* bandsテーブル
  
| id | band_name | act_Announcement_Date |
|------------|------------|------------|
| 1      | ASIAN KUNG-FU GENERATION      | 2023-09-01      |
| 2      | Rhythmic Toy World      | 2023-10-01      |
| 3      | UNISON SQUARE GARDEN      | 2023-11-01      |

* membersテーブル　※今回使用なし。
  
| id | member_name | part  | band_id |
|----|-------------|-------|---------|
|  1 | Gotoh       | Gt&Vo |       1 |
|  2 | Kita        | Gt    |       1 |
|  3 | Yamada      | Ba    |       1 |
|  4 | Izichi      | Drum  |       1 |
|  5 | Uchida      | Gt&Vo |       2 |
|  6 | Kishi       | Gt    |       2 |
|  7 | Sudo        | Ba    |       2 |
|  8 | Isomura     | Drum  |       2 |
|  9 | Saitoh      | Gt&Vo |       3 |

### ■動作確認(GET)

### ①bandsテーブルよりact_Announcement_Dateを除いたidとband_nameをReadする。  
```
curl --location 'http://localhost:8080/bands/all/names'
```
<img width="1245" alt="スクリーンショット 2023-08-27 16 42 10" src="https://github.com/ABECKCROW/lecture09/assets/136610341/335674c4-233c-4e86-bb3f-c51624c7d768">

*** 

### ②act_Announcement_Dateが今日以前の①をReadする。
```
curl --location 'http://localhost:8080/bands/announced/names'
```

<img width="1243" alt="スクリーンショット 2023-08-27 16 40 14" src="https://github.com/ABECKCROW/lecture09/assets/136610341/50fed30b-47f7-4f7c-903e-174dcaae3cde">

*** 

### ■動作確認(Create)
```
curl --location 'localhost:8080/bands/create' \
--header 'Content-Type: application/json' \
--data '{
    "bandName": "Official髭男dism",
    "actAnnouncementDate": "2023-12-01T00:00:00.000Z"
}
'
```

<img width="1247" alt="スクリーンショット 2023-09-08 17 54 16" src="https://github.com/ABECKCROW/lecture09/assets/136610341/0206c233-932c-411e-a589-bf21fb76919f">

<img width="1247" alt="スクリーンショット 2023-09-08 17 56 34" src="https://github.com/ABECKCROW/lecture09/assets/136610341/3da45275-6ef4-4ba8-bf21-7aa02edbda9d">

*** 


### ■動作確認(Patch)
```
curl --location --request PATCH 'localhost:8080/bands/update/9' \
--header 'Content-Type: application/json' \
--data '{
    "bandName": "ひげだん",
    "actAnnouncementDate": "2023-12-01T00:00:00.000Z"
}
'
```
<img width="1244" alt="スクリーンショット 2023-09-08 18 01 45" src="https://github.com/ABECKCROW/lecture09/assets/136610341/a8c74850-feab-436c-9d74-35b1d296dc75">

<img width="1246" alt="スクリーンショット 2023-09-08 18 02 49" src="https://github.com/ABECKCROW/lecture09/assets/136610341/e3ebc90c-3df1-4f26-9d3b-8cc105af407e">

*** 


### ■動作確認(Delete)
```
curl --location --request DELETE 'http://localhost:8080/bands/delete/9'
```
<img width="1244" alt="スクリーンショット 2023-09-08 18 03 49" src="https://github.com/ABECKCROW/lecture09/assets/136610341/cb66fea9-ffcf-4f5f-92dc-2f1b7a38ce82">

<img width="1241" alt="スクリーンショット 2023-09-08 18 04 29" src="https://github.com/ABECKCROW/lecture09/assets/136610341/f3a2643e-247b-418a-8fcd-131319745c0a">

*** 

### ■今後の展望
■お客さん側の利用想定
出演の発表がされたバンドを取得する。
出演発表がされていないバンドは取得できない。バンド情報も合わせて限定的に取得する。
そのバンドから参戦する自分だけのタイムテーブルを作る。ただし出演時間と移動時間が被る場合は重複している旨も合わせてレスポンスする。
移動時間に関しては最短の道で計算する。

■運営側の利用想定
出演決定した時点でバンド情報を作成する。合わせて発表日も設定。
出演バンドの発表前日にリマインドメールをする。

■やることリスト
* 最初はフェスを想定していなかったため、ネーミングを見直す。
* 出演バンド全取得、更新、削除は認証制にする。(Spring Security)
* 出演バンドの詳細情報を絞ってレスポンスしていきたい。
* バンドの登録時にメンバー人数分の登録処理をまとめたい。(トランザクション) 
* 更新できるデータは限定したい。

* 下記、テーブルの追加

| event_year	<br> | event_first_date | event_second_date | start_time   | end_time | 
| --------------- | ---------------- | ----------------- | ------------ | -------- | 
| 2024            | 08-03            | 08-04             | 10:00:00     | 20:00:00 | 
| 2025            |                  |                   |              |          | 
| 2026            |                  |                   |              |          | 

| stage_id<br> | stage_name | event_year | 
| ------------ | ---------- | ---------- | 
| １           | GRASS      | 2024       | 
| ２           | PARK       | 2024       | 
| ３           | LAKE       | 2024       | 

| stage_id<br> | stage_name   | from_stage_id | to_stage_id | travel_time | 
| ------------ | ------------ | ------------- | ----------- | ----------- | 
| １           | そよかぜ     | 1             | 2           | 10          | 
| ２           | そよかぜ     | 2             | 1           | 10          | 
| ３           | ぴょんぴょん | 2             | 3           | 10          | 
| 4            | ぴょんぴょん | 3             | 2           | 10          | 
| 5            | まつかぜ     | 3             | 1           | 20          | 
| 6            | まつかぜ     | 1             | 3           | 20          | 

| id  | band_name                | number_of_members | act_Announcement_Date | performance_day | start_time | end_time | stage_name | arrival_time<br>(出演時間の3時間前で自動計算) | 
| --- | ------------------------ | ----------------- | --------------------- | --------------- | ---------- | -------- | ---------- | --------------------------------------------- | 
| 1   | ASIAN KUNG-FU GENERATION |                   | 2023-09-01            |                 |            |          |            |                                               | 
| 2   | Rhythmic Toy World       |                   | 2023-10-01            |                 |            |          |            |                                               | 
| 3   | UNISON SQUARE GARDEN     |                   | 2023-11-01            |                 |            |          |            |                                               | 
