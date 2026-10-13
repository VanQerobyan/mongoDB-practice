# MongoDB Assignment
# Task 6 — MongoDB Installation and Connection

Տեղադրել եմ MongoDB Community Server-ը և MongoDB Shell-ը։

- **MongoDB version:** `8.0.32`
- **Mongosh version:** `2.13.0`

Ստուգել եմ MongoDB-ի աշխատանքը հետևյալ հրամանով՝

db.runCommand({ ping: 1 })

Ստացել եմ `{ ok: 1 }`, ինչը հաստատում է, որ MongoDB Server-ը ճիշտ է աշխատում։

`show dbs` հրամանով ստուգել եմ տվյալների բազաները։ Ցուցադրվել են `admin`, `config` և `local` տվյալների բազաները։

`admin` database-ը հիմնականում օգտագործվում է administration, authentication և authorization գործողությունների համար։
`config` database-ը MongoDB-ն օգտագործում է իր ներքին համակարգի կառուցվածքի ու աշխատանքի մասին տվյալներ պահելու համար:
`local` database-ը օգտագործվում է այն տվյալների համար, որոնք վերաբերում են միայն տվյալ կոնկրետ mongod server-ին։

## Task 7 — Admin User and Authentication

`admin` տվյալների բազայում ստեղծել եմ `adminUser` օգտատերը հետևյալ `roles`-ով՝

- `userAdminAnyDatabase`
- `readWriteAnyDatabase`

`/etc/mongod.conf` ֆայլում միացրել եմ `authorization`-ը՝

security:
  authorization: enabled

միացել եմ `adminUser`-ով՝

mongosh -u adminUser -p --authenticationDatabase admin

`show users` և `db.getUsers()` հրամաններով ստուգել եմ ստեղծված օգտատիրոջ տվյալներն ու նրա `roles`-ը։


`adminUser`-ը ստեղծել եմ մինչև authorization-ը միացնելը, որովհետև հակառակ դեպքում MongoDB-ն արդեն authentication կպահանջեր: Այսպիսով կարող էի հայտնվել այնպիսի վիճակում, երբ authorization-ը միացված է, բայց համապատասխան administrator user դեռ գոյություն չունի։ MongoDB-ն ունի նաև `localhost exception`, որը որոշ պայմաններում թույլ է տալիս localhost-ից ստեղծել առաջին administrator user-ը, եթե համակարգում դեռ ոչ մի user գոյություն չունի։

## Task 8 — Application User and Permissions

`bookstore` տվյալների բազայում ստեղծել եմ `appUser` օգտատերը տրամադրելով նրան  այդ տվյալների բազայի համար նախատեսված `readWrite` դերը։

Օգտատիրոջ միջոցով միացել եմ MongoDB-ին՝

mongosh -u appUser -p --authenticationDatabase bookstore


Օգտագործել եմ `--authenticationDatabase bookstore`, քանի որ `appUser`-ը ստեղծել եմ հենց `bookstore` database-ում, և MongoDB-ն այդ database-ում է ստուգում տվյալ user-ի credentials-ը։

`bookstore.books` collection-ում `insertOne()` մեթոդով ավելացրել եմ մեկ փաստաթուղթ, այնուհետև `findOne()` մեթոդով ստուգել եմ դրա առկայությունը։

Ստուգելու համար, որ `appUser`-ն ունի միայն իրեն տրամադրված թույլտվությունները, փորձել եմ document ավելացնել `other` տվյալների բազայում։ MongoDB-ն վերադարձրել է `Unauthorized` սխալ, քանի որ այդ տվյալների բազայում գրելու թույլտվություն չունեմ։

`adminUser`-ը երևում է `admin` database-ում, իսկ `appUser`-ը՝ `bookstore` database-ում։ Սա ցույց է տալիս, որ MongoDB user-ը կապված է այն database-ի հետ, որտեղ ստեղծվել է, և այդ database-ը հանդիսանում է նրա authentication database-ը։

## Task 9 — CRUD Operations

### 1. Insert Documents

`insertMany()` մեթոդով `books` collection-ում ավելացրել եմ չորս գիրք՝

- `MongoDb` — `price: 35`, `isPublished: 2022`, `tags: ['database', 'beginner']`
- `Node.js` — `isPublished: 2020`
- `React` — `price: 50`, `isPublished: 2019`
- `Clean Code` — միայն վերնագիրը

Collection-ում արդեն կար նաև `JavaScript` գիրքը՝ `price: 50` արժեքով։

### 2. Find, Sort and Limit

`find()` մեթոդով գտել եմ այն գրքերը, որոնց `price`-ը մեծ է `15`-ից։ `sort()` մեթոդով դասավորել եմ արդյունքները գնի նվազման կարգով, իսկ `limit()` մեթոդով սահմանափակել եմ արդյունքները երեք փաստաթղթով։

db.books.find({ price: { $gt: 15 } }).sort({ price: -1 }).limit(3)

### 3. Update Document

`updateOne()` մեթոդի և `$set` օպերատորի միջոցով փոխել եմ `JavaScript` գրքի գինը՝ `50`-ից դարձնելով `21`։

Արդյունքում ստացել եմ `matchedCount: 1` և `modifiedCount: 1`, ինչը ցույց է տվել, որ մեկ փաստաթուղթ փոփոխվել է։ `findOne()` մեթոդով ստուգել եմ փոփոխված տվյալը։

### 4. Delete Document

`deleteOne()` մեթոդով ջնջել եմ `JavaScript` գիրքը։

Արդյունքում ստացել եմ `deletedCount: 1`, ինչը հաստատել է, որ մեկ փաստաթուղթ ջնջվել է։


### 5. Collection Creation

books collection-ը ստեղծվեց ավտոմատ կերպով այն ժամանակ, երբ առաջին անգամ document insert արեցի դրա մեջ։ Ես առանձին createCollection() գործողություն չէի կատարել։ Սա ցույց է տալիս, որ MongoDB-ն կարող է collection ստեղծել առաջին write operation-ի ժամանակ: