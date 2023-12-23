---
title: "祇園祭WebサイトのDBについて"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "prisma", "postgresql", "excel", "powershell"]
published: true
---

## なにこれ
この記事は [木更津高専 Advent Calender 2023](https://qiita.com/advent-calendar/2023/nit_kisarazu) 参加記事です．
前日の記事は [@naotiki](https://qiita.com/naotiki) さんによる [学園祭の企画管理システム(一部)と認証システムを作った話](https://qiita.com/naotiki/items/6506ef15a50f1c49b981) でした．
投稿が遅れてしまったので，シリーズ2として [@toreis](https://qiita.com/toreis) さんによる [AtCoder入灰記事](https://qiita.com/toreis/items/b4d08ebf1ce1c3d8965f) が投稿されています．ありがとうございます．

今年度，所属しているプログラミング研究同好会(プロ研)のプロジェクトとして，学園祭(祇園祭)のWebサイトを制作しました．
私はDBおよびサーバ関係を担当したので，この記事では，DBにデータを渡す方法についてまとめておこうと思います．

## 使用した技術など
フレームワークには[Next.js](https://nextjs.org/)を選択し，ORMは[Prisma](https://www.prisma.io/)，DBには[PostgreSQL](https://www.postgresql.org/)を使用しました．

## SeedデータをPrismaに渡したい
学生・教員の所属情報や教室名などをSeedデータとしてPrismaに渡し，DBにデータを登録しました．

### Teams→JSON
学生・教員の所属情報を取得するため，Teamsのグループからメンバ一覧を取得し，JSON形式として吐き出すPowerShellスクリプトを書きました．
`MicrosoftTeams`モジュールを用いてTeamsに接続し，`ConvertTo-JSON`でオブジェクトをJSON形式に変換します．

```powershell
# Teamsにログインする
Connect-MicrosoftTeams | Out-Null

# グループ からメンバ一覧を取得する
$memberList = Get-TeamUser -GroupId <グループID> | ConvertTo-Json

# JSONに書き出す
$thisYear = Get-Date -UFormat %Y
$memberList | Out-File "./input/memberList${thisYear}.json" -Encoding utf8
```

### Excel→JSON
企画の情報は運営の部署からExcelデータとして降ってくるので，テーブルをJSON形式として出力するOfficeスクリプトを使いました．
ほぼ[Microsoftによるサンプル](https://learn.microsoft.com/ja-jp/office/dev/scripts/resources/samples/get-table-data)を使ったので，気になる方は参照してください．

### JSON→DB
上に挙げたデータにある不備(名前の表記ゆれなど)は[jq](https://jqlang.github.io/jq/)を使いつつ手動で修正しました．~~かなり地獄でした．~~
他に，教室名などのデータは元となるデータが学生便覧しかないので，JSONを手書きしました．~~これも地獄だった...~~

そんなこんなで生成されたJSONデータをPrismaに渡し，DBに登録します．
例として，`Member`テーブルに学生・教員の情報を登録する関数を示します．

```ts
import fs from "node:fs";
import path from "node:path";
import { PrismaClient } from "@prisma/client";
import { MemberListInput } from "../types/input.ts";

export const prisma = new PrismaClient();

/**
 * Memberテーブルを生成する
 * @return {Promise<void>}
 */
export async function createMember(): Promise<void> {
  const jsonPath = path.resolve(__dirname, "../input/memberList2023.json");
  const memberListInput = (await JSON.parse(fs.readFileSync(jsonPath, "utf-8"))) as MemberListInput;

  for (let i = 0; i < memberListInput.length; i++) {
    await prisma.member.create({
      data: {
        email: memberListInput[i].User,
        alias: memberListInput[i].User.split("@")[0],
        displayName: memberListInput[i].Name,
      },
    });
  }
}
```

## 最後に
このプロジェクトは，一緒に頑張ったプロ研の仲間たちや先輩方に助けられ，結果的になんとか成功となりました．
本当にありがとうございました．
