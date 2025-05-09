# ステップ3のクエリ一覧　　

1.よく見られているエピソードを知りたいです。  
エピソード視聴数トップ3のエピソードタイトルと視聴数を取得してください

```
SELECT
  title,
  total_view_count
FROM
  episodes
ORDER BY
  total_view_count DESC
LIMIT 3;
```

2.よく見られているエピソードの番組情報やシーズン情報も合わせて知りたいです。  
エピソード視聴数トップ3の番組タイトル、シーズン数、エピソード数、エピソードタイトル、視聴数を取得してください

```
SELECT
  p.title program_title,
  s.season_number,
  e.episode_number,
  e.title episode_title,
  e.total_view_count
FROM
  episodes e
INNER JOIN 
  programs p ON e.program_id = p.program_id
LEFT JOIN
  seasons s ON e.season_id = s.season_id
ORDER BY
  e.total_view_count DESC
LIMIT 3;
```
3.本日の番組表を表示するために、本日、どのチャンネルの、何時から、何の番組が放送されるのかを知りたいです。  
本日放送される全ての番組に対して、チャンネル名、放送開始時刻(日付+時間)、放送終了時刻、シーズン数、  
エピソード数、エピソードタイトル、エピソード詳細を取得してください。  
なお、番組の開始時刻が本日のものを本日方法される番組とみなすものとします

```
SELECT
    c.name AS channel_name,
    b.start_time,
    b.end_time,
    s.season_number,
    e.episode_number,
    e.title AS episode_title,
    e.description AS episode_description
FROM
    broadcast_slots b
INNER JOIN
    channels c ON b.channel_id = c.channel_id
INNER JOIN
    episodes e ON b.episode_id = e.episode_id
LEFT JOIN
    seasons s ON e.season_id = s.season_id
WHERE
    DATE(b.start_time) = CURRENT_DATE
ORDER BY
    channel_name, b.start_time; 
```
（サンプルデータで本日のデータを追加して行なったため、同じコードを実行しても動かない可能性大）

4.ドラマというチャンネルがあったとして、  
ドラマのチャンネルの番組表を表示するために、  
本日から一週間分、何日の何時から何の番組が放送されるのかを知りたいです。  
ドラマのチャンネルに対して、放送開始時刻、放送終了時刻、シーズン数、エピソード数、  
エピソードタイトル、エピソード詳細を本日から一週間分取得してください。


```
```

5.直近一週間で最も見られた番組が知りたいです。  
直近一週間に放送された番組の中で、エピソード視聴数合計トップ2の番組に対して、  
番組タイトル、視聴数を取得してください

```
```
6.ジャンルごとの番組の視聴数ランキングを知りたいです。  
番組の視聴数ランキングはエピソードの平均視聴数ランキングとします。  
ジャンルごとに視聴数トップの番組に対して、ジャンル名、番組タイトル、  
エピソード平均視聴数を取得してください。

```
```
