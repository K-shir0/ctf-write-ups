## Write-Up
192.168.111.0出ない人はアクセスできないと表示されるので、以下のヘッダーを付けて送信するとIPが偽装できる
```
X-Forwarded-For: 192.168.111.1 
```

アクセスできた内部サイトを観てみると
- htmlのselectにFlagが存在する。
- flagを選択してPOSTするとブロックされる、フォーマットは{"id": 数字}

"/"にFlagである{"id": 2}でPOST送信するとブロックされるたので、bff/main.goのソースコードを読むと、IDの2はバリデーションされている。
```
if info.ID == 2 {
			c.JSON(400, gin.H{"error": "It is forbidden to retrieve Flag from this BFF server."})
			return
}
```

その下の処理を読むとbffに送信されたjsonを再びボディにしてapiサーバ送っている処理が確認できる。
```
req, err := http.NewRequest("POST", "http://api:8000", bytes.NewReader(body))
```

/ に POST するとバリデーション処理は後ろのidが優先されて読み出され回避することが出来る。
```
{"id": 2, "id": 1}
```

その後、jsonをbodyにしapiを送信しているので、送った先のapiの処理が内部的に以下のようにになりFlagを取得できる。
```
{"id": 2}
```

## Flag
ctf4b{j50n_is_v4ry_u5efu1_bu7_s0metim3s_it_bi7es_b4ck}

