pkg - a package manager for FreeBSD
====================================


Motivation:
-----------

目標： lang/python311 に依存しているパッケージを lang/python312 に移行するために、明示的にインストールしたパッケージの一覧を調査するためです。
この一覧に基づいて pkg delete -y lang/python311 を実行、その後 lang/python312 に依存する状態で、必要最小限のパッケージをインストールを目指します。

Objective: To identify explicitly installed packages that depend on lang/python311 in order to migrate them to lang/python312.
Based on this list, the plan is to run pkg delete -y lang/python311 and then reinstall the minimal set of packages built against lang/python312.

現状： この一覧は極単純に、下記のコマンドを実行すれば得られます。

Current Situation: This list can be obtained simply by running the following command via pkg shell:

```
pkg shell "SELECT p.origin FROM packages AS p JOIN deps AS d ON p.id = d.package_id WHERE p.automatic = 0 AND d.origin = 'lang/python311'"
```

このクエリーは pkg query -e では一発で実行できません。下記のように awk を組み合わせる必要があります。

However, this cannot be executed directly using a single pkg query -e command. It currently requires piping to awk as shown below:

```
pkg query -e "%a = 0" "%do %o" | awk '/^lang\/python311/{ print $2 }'
```

これは EVALUATION FORMAT において、`%do` を指定して問い合わせすることができないためです。
そこで `in` オペレーターを追加してこれらの調査ができるよう改修します。

This is because `%do` cannot be used within the EVALUATION FORMAT string.
Proposal: Therefore, I propose extending the EVALUATION FORMAT by adding an `in` operator to enable this type of query natively.

Status:
-------

AI generated; not reviewed by humans
