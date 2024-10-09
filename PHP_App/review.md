# PHP App ① レビュー

## 全般

### 以下のaタグのリンクを押下した際にedit.phpの$_GETにどんな値が格納されるか説明してください。

```html
<a href="edit.php?todo_id=123&todo_content=焼肉">更新</a>
```

### 以下のフォームの送信ボタンを押下した際にstore.phpの$_POSTにどんな値が格納されるか説明してください。

```html
<form action="store.php" method="post">
    <input type="text" name="id" value="123">
		<textarea　name="content">焼肉</textarea>
    <button type="submit">送信</button>
</form>
```

### `require_once()` は何のために記述しているか説明してください。
- require_once式は指定されたファイルを読み込み、ファイルがすでに読み込まれているかどうかを PHP がチェックする。

### `savePostedData($post)`は何をしているか説明してください。
```php
function getRefererPath()
{
    $urlArray = parse_url($_SERVER['HTTP_REFERER']);
    // var_dump($urlArray);
    // exit();
    return $urlArray['path'];
}
```
savePostedData($post)関数は、上記の関数（リクエスト元のURLを文字列で取得しそのパスを返す）を呼び出し、リクエスト元のURLのパスを条件分岐をして処理を振り分けている。
### `header('location: ./index.php')`は何をしているか説明してください。
- eader関数指定したファイルに遷移している。

### `getRefererPath()`は何をしているか説明してください。
- リクエスト元のURLを文字列で取得し、そのパスを返している。
### `connectPdo()` の返り値は何か、またこの記述は何をするための記述か説明してください。
- 返り値はnew PDO(DSN, DB_USER, DB_PASSWORD)
- 例外処理をするための記述
### `try catch`とは何か説明してください。
- 例外処理をしている。tryの部分で例外を発生させるコードを記述し、catchで例外が発生した場合の処理を記述する。

### Pdoクラスをインスタンス化する際に`try catch`が必要な理由を説明してください。
- 例外が起きた時にも正しく動くように対処するため。
## 新規作成

### `createTodoData($post)`は何をしているか説明してください。
- connection.phpでまとめたDBへの操作にデータを渡している。
## 一覧

### `getTodoList()`の返り値について説明してください。
- 戻り値はgetAllRecords()でPDOクラスのメソッド。その戻り値が$dbh->query($sql)->fetchAll()
### `<?= ?>`は何の省略形か説明してください。
- echo
## 更新

### `getSelectedTodo($_GET['id'])`の返り値は何か、またなぜ`$_GET['id']` を引数に渡すのか説明してください。
```php
function getSelectedTodo($id)
{
    return getTodoTextById($id); 
}
```
- `getSelectedTodo($_GET['id'])`の返り値は、`getTodoTextById($id)`です。
- `$_GET['id']` を引数に渡す理由は、`getTodoTextById($id)`関数（返り値）で現在保存されているTODOの内容を返す処理をし、
  その内容を $todoに格納しているから。

### `updateTodoData($post)`は何をしているか説明してください。
```php
function updateTodoData($post)//更新
{
    $dbh = connectPdo();
    $sql = 'UPDATE todos SET content = "' . $post['content'] . '" WHERE id = ' . $post['id'];
    $dbh->query($sql);
}
```
$sqlでDBに登録済みのデータの更新する処理を行い、queryメソッドでSQL文を実行している。

## 削除

### `deleteTodoData($id)`は何をしているか説明してください。
```php
function deleteTodoData($id)
{
    $dbh = connectPdo();
    $now = date('Y-m-d H:i:s');
    $sql = "UPDATE todos SET deleted_at = '$now' WHERE id = $id";//""で文字列としているから中に''しても変数展開はされる。文字列結合と同等の扱いがある。
    $dbh->query($sql);
}
```
`deleteTodoData($id)`はDBからデータを論理削除する処理を行っている。
手順は以下の通りです。
まずdate関数を使い現在時刻を取得し$nowに代入をする。
次に指定したidのdeleted_atレコードのカラムを現在時刻に変更する処理をし、
そのsql文を実行している。

### `deleted_at`を現在時刻で更新すると一覧画面からToDoが非表示になる理由を説明してください。
```php
function getAllRecords()//関数を定義しているだけでこれだけでは使えない（呼び出してないから）一覧
{
    $dbh = connectPdo();
    $sql = 'SELECT * FROM todos WHERE deleted_at IS NULL'; //*は全カラム

    return $dbh->query($sql)->fetchAll();
    //$dbh->query($sql)でDBに対して作成したSQL文で実行し、fetchall()で実行結果を全配列で取得
}//query（PDOクラスの）メソッドの戻り値はPDOstaementオブジェクト。PDOstaementクラスのメソッドがfetchAll
```
上記の関数の`$sql = 'SELECT * FROM todos WHERE deleted_at IS NULL'`でtodosテーブルのdeleted_atがNULLのものに絞って全カラムを取得する処理をしているから`deleted_at`を現在時刻で更新すると一覧画面からToDoが非表示になる。

### 今回のように実際のデータを削除せずに非表示にすることで削除されたように扱うことを〇〇削除というか。
- 論理削除

### 実際にデータを削除することを〇〇削除というか。
- 物理削除

### 前問のそれぞれの削除のメリット・デメリットについて説明してください。
- 論理削除のメリット

  DBから直接データを消しているわけでないためデータを事実上残しておける。
  復元が可能

- 論理削除のデメリット

  DBにデータが残り続けるためDB容量を圧迫する。

- 物理削除のメリット

  不要なデータを完全に削除することができ、DBが整理される。

- 物理削除デメリット
  
  完全にデータを削除しているため復旧ができない。
  