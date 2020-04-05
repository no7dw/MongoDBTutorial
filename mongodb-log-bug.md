title: mongodb-log-bug
date: 2016-01-25 22:33:08
tags:
---
mongoshell

    > db.user_account_cashflow.remove({ user_id: "5585065a34a4f724f21894f9", objId: "569d653911a64a587b645b94", type: 1 })
    WriteResult({ "nRemoved" : 1 })
    > 
    
    
    > db.user_account_cashflow.update( { user_id: "5585065a34a4f724f21894f9", objId: "569d653911a64a587b645b94", type: 1 }, { $set: { time: 1453155641285.0, amount: 3000, type: 1, user_id: "5585065a34a4f724f21894f9", objId: "569d653911a64a587b645b94", order_id: "569d653911a64a587b645b94", remark: "购买xx理财3000元", product_id: "549922452238c54e98b750bc", asset_id: "561d38143be1c145fac00f59", transactions_ids: [ "569d655611a64a587b645b96" ] } } , {upsert:1} )
    WriteResult({
        "nMatched" : 0,
        "nUpserted" : 1,
        "nModified" : 0,
        "_id" : ObjectId("56a62fb37b4d214b2f5e79f2")
    })

//journal

    2016-01-25T22:22:43.604+0800 I WRITE    [conn4] update new_koala.user_account_cashflow query: { user_id: "5585065a34a4f724f21894f9", objId: "569d653911a64a587b645b94", type: 1.0 } update: { $set: { time: 1453155641285.0, amount: 3000.0, type: 1.0, user_id: "5585065a34a4f724f21894f9", objId: "569d653911a64a587b645b94", order_id: "569d653911a64a587b645b94", remark: "购买xx理财3000元", product_id: "549922452238c54e98b750bc", asset_id: "561d38143be1c145fac00f59", transactions_ids: [ "569d655611a64a587b645b96" ] } } keysExamined:433 docsExamined:433 nMatched:1 nModified:1 upsert:1 keyUpdates:0 writeConflicts:0 numYields:3 locks:{ Global: { acquireCount: { r: 4, w: 4 } }, Database: { acquireCount: { w: 4 } }, Collection: { acquireCount: { w: 4 } } } 1ms
    2016-01-25T22:22:43.604+0800 I COMMAND  [conn4] command new_koala.$cmd command: update { update: "user_account_cashflow", updates: [ { q: { user_id: "5585065a34a4f724f21894f9", objId: "569d653911a64a587b645b94", type: 1.0 }, u: { $set: { time: 1453155641285.0, amount: 3000.0, type: 1.0, user_id: "5585065a34a4f724f21894f9", objId: "569d653911a64a587b645b94", order_id: "569d653911a64a587b645b94", remark: "购买xx理财3000元", product_id: "549922452238c54e98b750bc", asset_id: "561d38143be1c145fac00f59", transactions_ids: [ "569d655611a64a587b645b96" ] } }, multi: false, upsert: true } ], ordered: true } keyUpdates:0 writeConflicts:0 numYields:0 reslen:91 locks:{ Global: { acquireCount: { r: 4, w: 4 } }, Database: { acquireCount: { w: 4 } }, Collection: { acquireCount: { w: 4 } } } protocol:op_command 1ms


////////////////////////////////////////////////////////////////////////
//  BUG: why   nMatched:1
////////////////////////////////////////////////////////////////////////

========================
update at second time 
mongoshell

    > db.user_account_cashflow.update( { user_id: "5585065a34a4f724f21894f9", objId: "569d653911a64a587b645b94", type: 1 }, { $set: { time: 1453155641285.0, amount: 3000, type: 1, user_id: "5585065a34a4f724f21894f9", objId: "569d653911a64a587b645b94", order_id: "569d653911a64a587b645b94", remark: "购买xx理财3000元", product_id: "549922452238c54e98b750bc", asset_id: "561d38143be1c145fac00f59", transactions_ids: [ "569d655611a64a587b645b96" ] } } , {upsert:1} )
    WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 0 })
    > 


//journal



    2016-01-25T22:22:48.283+0800 I WRITE    [conn4] update new_koala.user_account_cashflow query: { user_id: "5585065a34a4f724f21894f9", objId: "569d653911a64a587b645b94", type: 1.0 } update: { $set: { time: 1453155641285.0, amount: 3000.0, type: 1.0, user_id: "5585065a34a4f724f21894f9", objId: "569d653911a64a587b645b94", order_id: "569d653911a64a587b645b94", remark: "购买xx理财3000元", product_id: "549922452238c54e98b750bc", asset_id: "561d38143be1c145fac00f59", transactions_ids: [ "569d655611a64a587b645b96" ] } } keysExamined:434 docsExamined:434 nMatched:1 nModified:0 keyUpdates:0 writeConflicts:0 numYields:3 locks:{ Global: { acquireCount: { r: 4, w: 4 } }, Database: { acquireCount: { w: 4 } }, Collection: { acquireCount: { w: 4 } } } 1ms
    2016-01-25T22:22:48.283+0800 I COMMAND  [conn4] command new_koala.$cmd command: update { update: "user_account_cashflow", updates: [ { q: { user_id: "5585065a34a4f724f21894f9", objId: "569d653911a64a587b645b94", type: 1.0 }, u: { $set: { time: 1453155641285.0, amount: 3000.0, type: 1.0, user_id: "5585065a34a4f724f21894f9", objId: "569d653911a64a587b645b94", order_id: "569d653911a64a587b645b94", remark: "购买xx理财3000元", product_id: "549922452238c54e98b750bc", asset_id: "561d38143be1c145fac00f59", transactions_ids: [ "569d655611a64a587b645b96" ] } }, multi: false, upsert: true } ], ordered: true } keyUpdates:0 writeConflicts:0 numYields:0 reslen:40 locks:{ Global: { acquireCount: { r: 4, w: 4 } }, Database: { acquireCount: { w: 4 } }, Collection: { acquireCount: { w: 4 } } } protocol:op_command 1ms

----

    klg@koala:~$ mongod --version
    db version v3.2.1
    git version: a14d55980c2cdc565d4704a7e3ad37e4e535c1b2
    OpenSSL version: OpenSSL 1.0.1f 6 Jan 2014
    allocator: tcmalloc
    modules: none
    build environment:
        distmod: ubuntu1404
        distarch: x86_64
        target_arch: x86_64
    


