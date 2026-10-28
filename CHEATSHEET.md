# Load Data into DynamoDB — Cheat Sheet

> 📚 **Part 1 of 2** in my DynamoDB projects · Next: [Query Data with DynamoDB](https://github.com/jasahorro/Query-Data-with-DynamoDB/blob/main/CHEATSHEET.md) →

Lesson notes for reviewing later. The project write-up is in [README.md](README.md).

## Contents

1. [What I built](#what-i-built)
2. [Key terms](#key-terms)
3. [DynamoDB vs relational databases](#dynamodb-vs-relational-databases)
4. [Partition keys and sort keys](#partition-keys-and-sort-keys)
5. [Capacity: RCUs, WCUs and auto scaling](#capacity-rcus-wcus-and-auto-scaling)
6. [Working in the console](#working-in-the-console)
7. [CloudShell and the AWS CLI](#cloudshell-and-the-aws-cli)
8. [Creating tables with the CLI](#creating-tables-with-the-cli)
9. [Loading data with `batch-write-item`](#loading-data-with-batch-write-item)
10. [Build order and cleanup](#build-order-and-cleanup)
11. [Self-quiz](#self-quiz)

---

## What I built

- 🌟 **A DynamoDB table made in the console**, with student items (e.g. Nikko) and attributes like `ProjectsComplete`, set to the lowest capacity with auto scaling off to stay in the Free Tier.
- ⬆️ **Four tables made with the AWS CLI** in CloudShell: `ContentCatalog`, `Forum`, `Post` and `Comment`.
- 📦 **Sample data bulk-loaded** into all four from JSON files with `batch-write-item`.
- 🔎 **Items inspected and edited** in the console to see how each item can have its own attributes.

```
  AWS Console ──► CloudShell (AWS CLI built in, 1 GB storage)
                      │
                      │ create-table ×4, wait table-exists ×4
                      │ unzip sample data → *.json
                      │ batch-write-item --request-items file://X.json
                      ▼
  ┌────────────────┐ ┌────────┐ ┌───────────────────┐ ┌─────────────────────────┐
  │ ContentCatalog │ │ Forum  │ │ Post              │ │ Comment                 │
  │ PK: Id (N)     │ │ PK:Name│ │ PK: ForumName     │ │ PK: Id                  │
  │                │ │        │ │ SK: Subject       │ │ SK: CommentDateTime     │
  └────────────────┘ └────────┘ └───────────────────┘ └─────────────────────────┘
```

## Key terms

| Term | Plain meaning | Analogy |
|---|---|---|
| **DynamoDB** | AWS's fully managed, non-relational (NoSQL, key-value) database | A filing cabinet where every folder can hold different papers |
| **Non-relational / NoSQL** | Data isn't stored in fixed rows and columns, and you don't query it with SQL | Folders instead of a spreadsheet |
| **Table** | Where all DynamoDB data lives; a collection of items | The filing cabinet |
| **Item** | One record in a table (e.g. the student Nikko) | One folder |
| **Attribute** | One piece of data about an item (e.g. `ProjectsComplete`) | One sheet of paper in the folder |
| **Partition key** | The attribute DynamoDB uses to split up and find data; every item must have one | The label on the folder's tab |
| **Sort key** | An optional second key that orders items sharing a partition key | Sorting the folders behind one tab by date |
| **RCU / WCU** | Read and write capacity units: how many reads/writes per second the table can do | Engines powering the table |
| **Auto scaling** | Automatically raises or lowers capacity to match demand | A car that adds engines when it hits a hill |
| **CloudShell** | A terminal inside the AWS console, with the AWS CLI already installed | A ready-made workbench in the AWS office |
| **AWS CLI** | Software for creating, updating and deleting AWS resources with commands | Typing orders instead of clicking menus |

## DynamoDB vs relational databases

- A **relational** table needs every row to have the **same columns**. Add a `StudentsComplete` column and every row must have it, even when it doesn't apply.
- A **DynamoDB** table is a **list of items, each with its own list of attributes**. It's like a spreadsheet where every row can have a different number of columns with different headers.
- The console *shows* DynamoDB data as rows and columns, but don't let that fool you. Blank cells just mean that item doesn't have that attribute.

| | DynamoDB (non-relational) | Relational (e.g. Aurora, MySQL) |
|---|---|---|
| **Structure** | Items with their own attributes | Rows that all share the same columns |
| **Flexibility** | ✅ Items can look completely different (great for e.g. shop products of different kinds) | ❌ Changing the structure affects every row |
| **Speed** | ✅ Partition keys jump straight to the right data | Can slow down when queries have to search large tables |
| **Queries** | Simple lookups by key | ✅ SQL makes complex queries straightforward |
| **Data quality** | Up to your app | ✅ A strict schema keeps data precise and consistent (why e.g. healthcare records often use relational) |

## Partition keys and sort keys

- **Partition key** = the filter DynamoDB uses to **split up and find** data (e.g. "find all items whose Color is Green").
- **Every item must have a value for the partition key**. Otherwise DynamoDB would have no way of finding it.
- **Sort key** (optional) = a second key that orders items with the same partition key, e.g. `Post` uses `ForumName` + `Subject`.
- **Uniqueness:** the lesson says partition key values don't have to be unique. That's true **only when the table also has a sort key**. Then the *partition key + sort key* pair must be unique. In a table with only a partition key (like `ContentCatalog` or `Forum`), each partition key value must be unique.

| Table | Partition key (`HASH`) | Sort key (`RANGE`) |
|---|---|---|
| `ContentCatalog` | `Id` (number) | — |
| `Forum` | `Name` (string) | — |
| `Post` | `ForumName` (string) | `Subject` (string) |
| `Comment` | `Id` (string) | `CommentDateTime` (string) |

## Capacity: RCUs, WCUs and auto scaling

- **Why not use the defaults?** To stay inside the **Free Tier**. The project uses the lowest-cost settings.
- **Item read/second** = how many items your app can read each second. **Item write/second** = how many it can save or update each second.

| Unit | What 1 unit gives you | Analogy |
|---|---|---|
| **RCU** | Up to **2 reads/second** (for eventually consistent reads of items up to 4 KB; a strongly consistent read gets 1) | One engine doing reads. Want 3 reads/second? You need 2 engines (2 RCUs). |
| **WCU** | **1 write/second** (items up to 1 KB) | One engine doing writes, so writes cost more than reads |

- **Free Tier:** 25 GB of storage + **25 RCUs and 25 WCUs** of provisioned capacity, which is enough for about **200 million requests a month**.
- RCUs are a **speedometer, not a fuel tank**. The green banner saying "**0.5** capacity units consumed" doesn't mean you have 24.5 left. It's how much was used *that second*. As long as you don't ask for lots of operations in the same second, you're fine.
- **The cost estimate** on the setup page **ignores the Free Tier**. AWS checks your real usage before billing and doesn't charge you if you stayed inside the limits.
- **Auto scaling** adjusts capacity to real-time demand (EC2 has it too, for the number of instances). It's handy, but if nobody's watching it can boost the table **past the Free Tier**, so it's **turned off** here.

## Working in the console

- DynamoDB lets you **add and edit items directly in the console**. Almost no other AWS database service does this (Amazon Keyspaces is the only other one).
- That's because it's built to be beginner- and developer-friendly: change the data and instantly see how your app responds.
- It's also a sign that DynamoDB is **fully managed**. AWS handles availability, durability and scaling, and you focus on the app.

## CloudShell and the AWS CLI

- **CloudShell** = a shell inside the AWS console where you can run code. The **AWS CLI comes pre-installed**, so there's nothing to set up.
- **AWS CLI** = create, update and delete AWS resources with commands instead of clicking. The console is great for learning; the CLI is faster and can be scripted, which is why professionals rely on it.
- CloudShell has **1 GB of persistent storage**, so files you unzip or scripts you write there stay between sessions.
- `cat Forum.json` prints a file's contents in the terminal, a quick way to check a file without opening it anywhere else.

## Creating tables with the CLI

```bash
aws dynamodb create-table \
    --table-name Post \
    --attribute-definitions \
        AttributeName=ForumName,AttributeType=S \
        AttributeName=Subject,AttributeType=S \
    --key-schema \
        AttributeName=ForumName,KeyType=HASH \
        AttributeName=Subject,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --query "TableDescription.TableStatus"
```

| Part | Meaning |
|---|---|
| `--attribute-definitions` | Names and types of the **key** attributes only: `S` = string, `N` = number |
| `--key-schema` | Which attribute is the partition key (`HASH`) and which is the sort key (`RANGE`) |
| `--provisioned-throughput` | 1 RCU and 1 WCU, to stay in the Free Tier |
| `--query "TableDescription.TableStatus"` | Only print the table's status (e.g. `CREATING`) instead of the whole description |

The same command runs for `ContentCatalog`, `Forum` and `Comment` with their own keys (see the table in [Partition keys and sort keys](#partition-keys-and-sort-keys)).

**Wait until the tables are ready:**

```bash
aws dynamodb wait table-exists --table-name ContentCatalog
aws dynamodb wait table-exists --table-name Forum
aws dynamodb wait table-exists --table-name Post
aws dynamodb wait table-exists --table-name Comment
```

- A **wait** command keeps the terminal waiting until a condition is met (here, until the table exists). Commands that depend on the table would fail otherwise.
- Run it *before* creating the resource and it keeps waiting until it decides the resource doesn't exist, then fails.

## Loading data with `batch-write-item`

**Inside a data file (`Forum.json`):**

```json
{
  "Forum": [
    { "PutRequest": { "Item": { "Name": {"S": "..."}, "Category": {"S": "..."}, "...": "..." } } },
    { "PutRequest": { "Item": { "...": "..." } } }
  ]
}
```

- `"Forum"` → which **table** the data goes into. That's how DynamoDB knows where each file's items belong.
- `"PutRequest"` → **add a new item**.
- The rest is the item's attributes. In `Forum.json` the first item has **five** attributes (Name, Category, Posts, Comments, Views) and the second only **three**: DynamoDB's flexibility in action.

**Load all four files:**

```bash
aws dynamodb batch-write-item --request-items file://ContentCatalog.json
aws dynamodb batch-write-item --request-items file://Forum.json
aws dynamodb batch-write-item --request-items file://Post.json
aws dynamodb batch-write-item --request-items file://Comment.json
```

| Part | Meaning |
|---|---|
| `batch-write-item` | Load many items in one go |
| `--request-items` | The items to write are in a file |
| `file://` | The file is stored locally, in CloudShell |

- **`UnprocessedItems`** in the output = records that **weren't written**. If it's not empty, something went wrong while loading.
- Afterwards, `ContentCatalog` in the console shows its partition key `Id` as the first column and **6 items**. Items like a project (Id 1) and a video (Id 203) have different attributes in the same table.

## Build order and cleanup

1. Create a table in the **console**: set the partition key, choose custom settings, set the lowest RCUs/WCUs, and **turn auto scaling off**.
2. Add items and attributes in the console.
3. Open **CloudShell**.
4. Run `create-table` for `ContentCatalog`, `Forum`, `Post` and `Comment`.
5. Run `wait table-exists` for each.
6. Download and **unzip** the sample data in CloudShell; check a file with `cat`.
7. Run `batch-write-item` for each JSON file and check `UnprocessedItems` is empty.
8. Explore and edit items in the console.

**Cleanup:** delete all the DynamoDB tables when you're done, so you're never charged. *(If you're going straight on to Query Data with DynamoDB, which uses the same four tables, keep them until you've finished that.)*

---

## Self-quiz

<details><summary>1. What makes DynamoDB a non-relational database?</summary>

It doesn't store data in fixed rows and columns. Each table is a set of items, and each item has its own set of attributes. It's also called NoSQL (not queried with SQL) or key-value.
</details>

<details><summary>2. What's the difference between an item and an attribute?</summary>

An **item** is one record (e.g. the student Nikko). An **attribute** is one piece of data about that item (e.g. `ProjectsComplete`).
</details>

<details><summary>3. What is a partition key, and why must every item have one?</summary>

The attribute DynamoDB uses to split up and find data. Without a value for it, DynamoDB would have no way of finding the item.
</details>

<details><summary>4. When can two items share the same partition key value?</summary>

Only when the table also has a **sort key**. Then the partition key + sort key pair must be unique (e.g. two posts in the same forum with different subjects).
</details>

<details><summary>5. Why did you change the default capacity settings and turn off auto scaling?</summary>

To stay in the Free Tier. Auto scaling can raise capacity past the Free Tier limits if nobody's watching it.
</details>

<details><summary>6. You want 3 reads per second. How many RCUs do you need, and why?</summary>

**2**. One RCU handles up to 2 (eventually consistent) reads per second, so 3 needs a second "engine."
</details>

<details><summary>7. Why do writes cost more than reads?</summary>

1 WCU gives 1 write per second, while 1 RCU gives up to 2 reads per second.
</details>

<details><summary>8. The console says 0.5 capacity units were consumed. Do you now have 24.5 of your 25 left?</summary>

No. Capacity units are a speedometer (how much per second), not a fuel tank. You're fine as long as you don't do lots of operations in the same second.
</details>

<details><summary>9. Why does the cost estimate on the setup page look scary?</summary>

It ignores the Free Tier. AWS checks your real usage before billing and won't charge you if you're inside the limits.
</details>

<details><summary>10. What is CloudShell, and why use it instead of your own terminal?</summary>

A terminal inside the AWS console with the AWS CLI pre-installed and already signed in, plus 1 GB of persistent storage. Nothing to install or configure.
</details>

<details><summary>11. In <code>create-table</code>, what do <code>HASH</code> and <code>RANGE</code> mean?</summary>

`HASH` = partition key, `RANGE` = sort key.
</details>

<details><summary>12. Why run <code>aws dynamodb wait table-exists</code> before loading data?</summary>

Tables take a moment to create. The wait command holds until each one exists, so the data-loading commands don't fail.
</details>

<details><summary>13. How does <code>batch-write-item</code> know which table to put the data in?</summary>

The top-level key in the JSON file names the table (e.g. `"Forum"`), and each `"PutRequest"` adds an item to it.
</details>

<details><summary>14. What does it mean if the output shows <code>UnprocessedItems</code>?</summary>

Some records weren't written to the table, because an error happened while loading.
</details>

<details><summary>15. Give one reason to choose a relational database over DynamoDB.</summary>

SQL makes complex queries straightforward, and the strict schema keeps data precise and consistent (e.g. healthcare patient records).
</details>
