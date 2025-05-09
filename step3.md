# ステップ3のクエリ一覧　　

サンプルコードを追加しないと動かないコードが多々あったので  
あくまでこういったコードで検索したものとしてご覧ください。


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


4.ドラマというチャンネルがあったとして、  
ドラマのチャンネルの番組表を表示するために、  
本日から一週間分、何日の何時から何の番組が放送されるのかを知りたいです。  
ドラマのチャンネルに対して、放送開始時刻、放送終了時刻、シーズン数、エピソード数、  
エピソードタイトル、エピソード詳細を本日から一週間分取得してください。


```
SELECT
   b.start_time,
   b.end_time,
   s.season_number,
   e.episode_number,
   e.title,
   e.description
FROM
  broadcast_slots b
INNER JOIN
  channels c ON b.channel_id = c.channel_id
INNER JOIN 
  episodes e ON b.episode_id = e.episode_id
LEFT JOIN
  seasons s ON e.season_id = s.season_id
WHERE
  c.name = 'ドラマコレクション'
  AND DATE(b.start_time) >= CURRENT_DATE
  AND DATE(b.start_time) < DATE_ADD(CURRENT_DATE, INTERVAL 7 DAY)
ORDER BY
  b.start_time;
```


5.直近一週間で最も見られた番組が知りたいです。  
直近一週間に放送された番組の中で、エピソード視聴数合計トップ2の番組に対して、  
番組タイトル、視聴数を取得してください

```
SELECT
  p.title program_title,
  SUM(bs.view_count) total_program_views
FROM
  programs p
INNER JOIN
  episodes e ON p.program_id = e.program_id
INNER JOIN
  broadcast_slots b ON e.episode_id = b.episode_id
INNER JOIN
  broadcast_slot_views bs ON b.slot_id = bs.slot_id
WHERE
  DATE(b.start_time) >= CURRENT_DATE
  AND DATE(b.start_time) < DATE_ADD(CURRENT_DATE, INTERVAL 7 DAY)
GROUP BY
  p.program_id,
  p.title
ORDER BY
  total_program_views DESC
LIMIT 2;
```


6.ジャンルごとの番組の視聴数ランキングを知りたいです。  
番組の視聴数ランキングはエピソードの平均視聴数ランキングとします。  
ジャンルごとに視聴数トップの番組に対して、ジャンル名、番組タイトル、  
エピソード平均視聴数を取得してください。

```
WITH ProgramAverageViews AS (
  SELECT
    p.program_id,
    p.title program_title,
    AVG(e.total_view_count) average_episode_views
  FROM
    programs p
  INNER JOIN
    episodes e ON p.program_id = e.program_id
  GROUP BY
    p.program_id,
    p.title
),
RankedProgramsByGenre AS (
  SELECT
    g.name genre_name,
    pav.program_title,
    pav.average_episode_views,
    ROW_NUMBER() OVER (PARTITION BY g.genre_id ORDER BY pav.average_episode_views DESC) rn
  FROM
    genres g
  INNER JOIN
    program_genres pg ON g.genre_id = pg.genre_id
  INNER JOIN
    ProgramAverageViews pav ON pg.program_id = pav.program_id
)
SELECT
  genre_name,
  program_title,
  average_episode_views
FROM
  RankedProgramsByGenre
WHERE
  rn = 1
ORDER BY
  genre_name;
```
