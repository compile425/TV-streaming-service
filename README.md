# internet-tv-service DB構築手順書

## 説明

好きな時間に好きな場所で話題の動画を無料で楽しめる  
「internet-tv-service」を作成するにあたって  
DBを構築するための手順書です。

この手順に沿っていくことで  
「インターネットTVサービス」に必要なDBの構築および  
サンプルデータの格納まで実行できます。

※この手順書ではMacでのMySQLの使用を想定しています。


## 手順１ MySQLのインストール

Homebrewを使ってインストールしていきます。

ターミナル内で下のコマンドを実行します。  
（一行ずつ実行するようにしてください。）

```
brew install mysql
brew services start mysql
```

その後は画面の指示に従い、  
エンターキーで進めていけばインストールは完了します。

途中でOSのパスワードが求められますが、  
入力が表示されないので注意してください。  
パスワードを入力したらエンターキーを押すことで進めることができます。

もし、Homebrewをインストールしてない場合は  
[Homebrew公式サイト](https://brew.sh/)からコマンドをターミナルで実行してください。

下記のようなコマンドが表示されているはずです。  
（更新されてる可能性もあるのでここの例と同じでなくても大丈夫です。）
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

その後、無事にHomebrewをインストールできたら下のコマンドを実行してください。
```
brew install mysql
brew services start mysql
```


## 手順2　セットアップファイルの作成

まずターミナルを開きます。  
'あなたのPC名'~ %　と表示されてるはずなので  
％のあとに下のコマンドを実行します。
```
cd ~/Desktop
```
```
'あなたのPC名'Desktop %
```
このように表示されていれば準備完了です。

次に下のコマンドを実行します。
```
cat << 'EOF_SQL' > set_up.sql
CREATE DATABASE IF NOT EXISTS internet_tv_service;

USE internet_tv_service;

CREATE TABLE genres (
    genre_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE
) ENGINE=InnoDB;

CREATE TABLE programs (
    program_id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    series_judgment BOOLEAN NOT NULL DEFAULT FALSE
) ENGINE=InnoDB;

CREATE TABLE program_genres (
    program_id INT NOT NULL,
    genre_id INT NOT NULL,
    PRIMARY KEY (program_id, genre_id),
    FOREIGN KEY (program_id) REFERENCES programs(program_id) ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (genre_id) REFERENCES genres(genre_id) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB;

CREATE TABLE seasons (
    season_id INT AUTO_INCREMENT PRIMARY KEY,
    program_id INT NOT NULL,
    season_number INT NOT NULL,
    UNIQUE (program_id, season_number),
    FOREIGN KEY (program_id) REFERENCES programs(program_id) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB;

CREATE TABLE episodes (
    episode_id INT AUTO_INCREMENT PRIMARY KEY,
    program_id INT NOT NULL,
    season_id INT NULL,
    episode_number INT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    duration_seconds INT,
    release_date DATE,
    FOREIGN KEY (program_id) REFERENCES programs(program_id) ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (season_id) REFERENCES seasons(season_id) ON DELETE SET NULL ON UPDATE CASCADE
) ENGINE=InnoDB;

CREATE TABLE channels (
    channel_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE
) ENGINE=InnoDB;

CREATE TABLE broadcast_slots (
    slot_id INT AUTO_INCREMENT PRIMARY KEY,
    channel_id INT NOT NULL,
    episode_id INT NOT NULL,
    start_time DATETIME NOT NULL,
    end_time DATETIME NOT NULL,
    UNIQUE (channel_id, start_time),
    FOREIGN KEY (channel_id) REFERENCES channels(channel_id) ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (episode_id) REFERENCES episodes(episode_id) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB;

CREATE TABLE broadcast_slot_views (
    slot_id INT NOT NULL PRIMARY KEY,
    view_count INT NOT NULL DEFAULT 0,
    FOREIGN KEY (slot_id) REFERENCES broadcast_slots(slot_id) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB;

INSERT INTO genres (genre_id, name) VALUES
(1, 'アニメ'),
(2, 'ドラマ'),
(3, '映画'),
(4, 'スポーツ'),
(5, 'ペット'),
(6, 'ニュース'),
(7, 'バラエティ');

INSERT INTO programs (program_id, title, description, series_judgment) VALUES
(1, '鬼滅の刃', '血風剣戟冒険譚！', TRUE),
(2, '今日の料理', '家庭でできる簡単レシピを紹介します。', FALSE),
(3, 'かわいいペット大集合！', '世界中のかわいいペットたちの映像をお届け！', TRUE),
(4, 'オーシャンズ11', '豪華キャストで贈るクライムエンターテイメント。', FALSE),
(5, 'ワールドカップ予選ハイライト', 'サッカーワールドカップアジア最終予選の激闘を振り返る。', FALSE),
(6, '週末お出かけ情報', '週末におすすめのスポットやイベントを紹介。', FALSE);

INSERT INTO program_genres (program_id, genre_id) VALUES
(1, 1),
(2, 7),
(2, 6),
(3, 5),
(3, 7),
(4, 3),
(5, 4),
(6, 7);

INSERT INTO seasons (season_id, program_id, season_number) VALUES
(1, 1, 1),
(2, 1, 2),
(3, 3, 1);

INSERT INTO episodes (episode_id, program_id, season_id, episode_number, title, description, duration_seconds, release_date) VALUES
(1, 1, 1, 1, '残酷', '炭治郎の家族が鬼に襲われる。', 1440, '2019-04-06'),
(2, 1, 1, 2, '育手・鱗滝左近次', '炭治郎は鱗滝左近次の元で修行を始める。', 1440, '2019-04-13'),
(3, 1, 2, 1, '音柱・宇髄天元', '遊郭潜入任務が始まる。', 1500, '2021-12-05'),
(4, 2, NULL, NULL, 'お手軽！鶏むね肉のハーブ焼き', 'パサつきがちな鶏むね肉をしっとり美味しく仕上げるコツを紹介。', 900, '2023-10-01'),
(5, 3, 3, 1, '初めてのお散歩', '子犬たちの初めてのお散歩に密着。', 1200, '2023-05-05'),
(6, 4, NULL, NULL, 'オーシャンズ11 (本編)', 'ラスベガスの巨大金庫を狙う11人のプロフェッショナル。', 7020, '2001-12-07'),
(7, 5, NULL, NULL, '日本 vs オーストラリア 激闘譜', '最終予選の天王山、日本対オーストラリア戦をダイジェストで。', 3600, '2023-03-24'),
(8, 6, NULL, NULL, '秋の紅葉特集！関東近郊おすすめスポット', '見頃を迎える紅葉の名所を厳選して紹介。', 1800, '2023-11-01');

INSERT INTO channels (channel_id, name) VALUES
(1, 'アニメシアター'),
(2, 'ドラマコレクション'),
(3, 'ムービープレミア'),
(4, 'スポーツライブ！'),
(5, '癒しのペットTV'),
(6, '情報バラエティCH');

INSERT INTO broadcast_slots (slot_id, channel_id, episode_id, start_time, end_time) VALUES
(1, 1, 1, '2024-05-20 20:00:00', '2024-05-20 20:24:00'),
(2, 1, 2, '2024-05-20 20:24:00', '2024-05-20 20:48:00'),
(3, 6, 4, '2024-05-21 12:00:00', '2024-05-21 12:15:00'),
(4, 5, 5, '2024-05-22 19:00:00', '2024-05-22 19:20:00'),
(5, 3, 6, '2024-05-25 21:00:00', '2024-05-25 22:57:00'),
(6, 4, 7, '2024-05-26 20:00:00', '2024-05-26 21:00:00'),
(7, 1, 1, '2024-05-27 10:00:00', '2024-05-27 10:24:00'),
(8, 6, 8, '2024-05-19 13:00:00', '2024-05-19 13:30:00');

INSERT INTO broadcast_slot_views (slot_id, view_count) VALUES
(1, 12500),
(2, 11800),
(3, 3200),
(4, 5500),
(5, 8900),
(6, 6700),
(7, 4500),
(8, 2800);
EOF_SQL
```
（コピーアンドペースト推奨）  
Desktop上にset_up.sqlのファイルが作成されていればOKです。


## 手順3 セットアップファイルの実行

まずはターミナル内で下のコマンドを実行し、MySQLに接続してください。
```
mysql -u root -p
```
パスワードを求められますが、エンターキーを押すだけで大丈夫です。
```
mysql>
```
このようにターミナル上の表記が変更されればOKです。

次に手順2で作成したセットアップファイルを実行します。  
以下のコマンドをターミナル内で実行してください。
```
SOURCE set_up.sql;
```
以下のコマンドをターミナル内で実行し、   
internet-tv-serviceが表示されれば成功です。
```
SHOW DATABASES;
```
成功例
```
+---------------------+
| Database            |
+---------------------+
| ec_site             |
| employees           |
| information_schema  |
| internet_tv_service |
| mysql               |
| performance_schema  |
| pokemon             |
| sys                 |
+---------------------+
```
他のDBも含まれてますが気にしないでください。  
internet_tv_serviceが含まれていればOKです。

もしこの手順の中でエラーが出た場合は落ち着いて同じ手順を繰り返してください。

タイピングミスの可能性もあるので手打ちでエラーが出た場合は  
手順書のコードをコピーアンドペーストしてみてください。

以上でDB構築は完了となります。  
セットアップファイルは削除しても構いません。  
お疲れ様でした。
