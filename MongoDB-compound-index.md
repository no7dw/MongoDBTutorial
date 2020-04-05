
根据典型碰到的场景，来做几个实验：
这里创建了个loans collection。简化只有100条数据。这个是借贷的表有 _id, userId, status(借贷状态), amount(金额). 

***看完 这个实验后， 你会明白了 {userId:1, status:1},  vs   {status:1,userId:1} 的差别***

PS：这个case 里面其实status 区分度不高，不应该建立的，这里只是作为实例展示。

总结：
 - 注意使用上 使用频率上 区分高的/常用的在前面
 - 如果需要减少索引以节省memory/提高修改数据的性能的话，可以保留区分度高，常用的，去除区分度不高，不常用的索引。

实验如下：

> db.loans.count()
100

> db.loans.find({ "userId" : "59e022d33f239800129c61c7", "status" : "repayed", }).explain()
{
  "queryPlanner" : {
    "namespace" : "cashLoan.loans",
    "parsedQuery" : {
      "$and" : [
        {
          "status" : {
            "$eq" : "repayed"
          }
        },
        {
          "userId" : {
            "$eq" : "59e022d33f239800129c61c7"
          }
        }
      ]
    },
    "winningPlan" : {
      "stage" : "COLLSCAN",
      "filter" : {
        "$and" : [
          {
            "status" : {
              "$eq" : "repayed"
            }
          },
          {
            "userId" : {
              "$eq" : "59e022d33f239800129c61c7"
            }
          }
        ]
      },
      "direction" : "forward"
    },
    "rejectedPlans" : [ ]
  },
  "ok" : 1
}

***注意上面 COLLSCAN 全表扫描了。因为没有索引。***
next 我们分别建立几个索引

***step 1 先建立 {userId:1, status:1}***


> db.loans.createIndex({userId:1, status:1})
{
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 1,
  "numIndexesAfter" : 2,
  "ok" : 1
}

> db.loans.find({ "userId" : "59e022d33f239800129c61c7", "status" : "repayed", }).explain()
{
  "queryPlanner" : {
    "namespace" : "cashLoan.loans",
    "parsedQuery" : {
      "$and" : [
        {
          "status" : {
            "$eq" : "repayed"
          }
        },
        {
          "userId" : {
            "$eq" : "59e022d33f239800129c61c7"
          }
        }
      ]
    },
    "winningPlan" : {
      "stage" : "FETCH",
      "inputStage" : {
        "stage" : "IXSCAN",
        "keyPattern" : {
          "userId" : 1,
          "status" : 1
        },
        "indexName" : "userId_1_status_1",
        "multiKeyPaths" : {
          "userId" : [ ],
          "status" : [ ]
        },
        "direction" : "forward",
        "indexBounds" : {
          "userId" : [
            "[\"59e022d33f239800129c61c7\", \"59e022d33f239800129c61c7\"]"
          ],
          "status" : [
            "[\"repayed\", \"repayed\"]"
          ]
        }
      }
    },
    "rejectedPlans" : [ ]
  },
  "ok" : 1
}

***如愿命中 {userId:1, status:1} 作为 winning plan***

***step2  再建立个典型的索引 userId***


> db.loans.createIndex({userId:1})
{
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 2,
  "numIndexesAfter" : 3,
  "ok" : 1
}

> db.loans.find({ "userId" : "59e022d33f239800129c61c7", "status" : "repayed", }).explain()
{
  "queryPlanner" : {
    "namespace" : "cashLoan.loans",
    "parsedQuery" : {
      "$and" : [
        {
          "status" : {
            "$eq" : "repayed"
          }
        },
        {
          "userId" : {
            "$eq" : "59e022d33f239800129c61c7"
          }
        }
      ]
    },
    "winningPlan" : {
      "stage" : "FETCH",
      "inputStage" : {
        "stage" : "IXSCAN",
        "keyPattern" : {
          "userId" : 1,
          "status" : 1
        },
        "indexName" : "userId_1_status_1",
        "multiKeyPaths" : {
          "userId" : [ ],
          "status" : [ ]
        },
        "direction" : "forward",
        "indexBounds" : {
          "userId" : [
            "[\"59e022d33f239800129c61c7\", \"59e022d33f239800129c61c7\"]"
          ],
          "status" : [
            "[\"repayed\", \"repayed\"]"
          ]
        }
      }
    },
    "rejectedPlans" : [
      {
        "stage" : "FETCH",
        "filter" : {
          "status" : {
            "$eq" : "repayed"
          }
        },
        "inputStage" : {
          "stage" : "IXSCAN",
          "keyPattern" : {
            "userId" : 1
          },
          "indexName" : "userId_1",
          "multiKeyPaths" : {
            "userId" : [ ]
          },
          "direction" : "forward",
          "indexBounds" : {
            "userId" : [
              "[\"59e022d33f239800129c61c7\", \"59e022d33f239800129c61c7\"]"
            ]
          }
        }
      }
    ]
  },
  "ok" : 1
}

***留意到 DB 检测到 {userId:1, status:1} 为更优执行的方案***

> db.loans.find({ "userId" : "59e022d33f239800129c61c7" }).explain()
{
  "queryPlanner" : {
    "namespace" : "cashLoan.loans",
    "parsedQuery" : {
      "userId" : {
        "$eq" : "59e022d33f239800129c61c7"
      }
    },
    "winningPlan" : {
      "stage" : "FETCH",
      "inputStage" : {
        "stage" : "IXSCAN",
        "keyPattern" : {
          "userId" : 1
        },
        "indexName" : "userId_1",
        "multiKeyPaths" : {
          "userId" : [ ]
        },
        "direction" : "forward",
        "indexBounds" : {
          "userId" : [
            "[\"59e022d33f239800129c61c7\", \"59e022d33f239800129c61c7\"]"
          ]
        }
      }
    },
    "rejectedPlans" : [
      {
        "stage" : "FETCH",
        "inputStage" : {
          "stage" : "IXSCAN",
          "keyPattern" : {
            "userId" : 1,
            "status" : 1
          },
          "indexName" : "userId_1_status_1",
          "multiKeyPaths" : {
            "userId" : [ ],
            "status" : [ ]
          },
          "direction" : "forward",
          "indexBounds" : {
            "userId" : [
              "[\"59e022d33f239800129c61c7\", \"59e022d33f239800129c61c7\"]"
            ],
            "status" : [
              "[MinKey, MaxKey]"
            ]
          }
        }
      }
    ]
  },
  "ok" : 1
}

***留意到 DB 检测到 {userId:1} 为更优执行的方案，嗯~，如我们所料***


> db.loans.find({ "status" : "repayed" }).explain()
{
  "queryPlanner" : {
    "namespace" : "cashLoan.loans",
    "parsedQuery" : {
      "status" : {
        "$eq" : "repayed"
      }
    },
    "winningPlan" : {
      "stage" : "COLLSCAN",
      "filter" : {
        "status" : {
          "$eq" : "repayed"
        }
      },
      "direction" : "forward"
    },
    "rejectedPlans" : [ ]
  },
  "ok" : 1
}

***有趣的部分：  status 不命中索引， 全表扫描 ***
接下来，我加了个sort 

> db.loans.find({ "userId" : "59e022d33f239800129c61c7" }).sort({status:1}).explain()
{
  "queryPlanner" : {
    "namespace" : "cashLoan.loans",
    "parsedQuery" : {
      "userId" : {
        "$eq" : "59e022d33f239800129c61c7"
      }
    },
    "winningPlan" : {
      "stage" : "FETCH",
      "inputStage" : {
        "stage" : "IXSCAN",
        "keyPattern" : {
          "userId" : 1,
          "status" : 1
        },
        "indexName" : "userId_1_status_1",
        "multiKeyPaths" : {
          "userId" : [ ],
          "status" : [ ]
        },
        "direction" : "forward",
        "indexBounds" : {
          "userId" : [
            "[\"59e022d33f239800129c61c7\", \"59e022d33f239800129c61c7\"]"
          ],
          "status" : [
            "[MinKey, MaxKey]"
          ]
        }
      }
    },
    "rejectedPlans" : [
      {
        "stage" : "SORT",
        "sortPattern" : {
          "status" : 1
        },
        "inputStage" : {
          "stage" : "SORT_KEY_GENERATOR",
          "inputStage" : {
            "stage" : "FETCH",
            "inputStage" : {
              "stage" : "IXSCAN",
              "keyPattern" : {
                "userId" : 1
              },
              "indexName" : "userId_1",
              "multiKeyPaths" : {
                "userId" : [ ]
              },
              "direction" : "forward",
              "indexBounds" : {
                "userId" : [
                  "[\"59e022d33f239800129c61c7\", \"59e022d33f239800129c61c7\"]"
                ]
              }
            }
          }
        }
      }
    ]
  },
  "ok" : 1
}

***有趣的部分：  status 不命中索引 ***


> db.loans.find({ "status" : "repayed","userId" : "59e022d33f239800129c61c7", }).explain()
{
  "queryPlanner" : {
    "namespace" : "cashLoan.loans",
    "parsedQuery" : {
      "$and" : [
        {
          "status" : {
            "$eq" : "repayed"
          }
        },
        {
          "userId" : {
            "$eq" : "59e022d33f239800129c61c7"
          }
        }
      ]
    },
    "winningPlan" : {
      "stage" : "FETCH",
      "inputStage" : {
        "stage" : "IXSCAN",
        "keyPattern" : {
          "userId" : 1,
          "status" : 1
        },
        "indexName" : "userId_1_status_1",
        "multiKeyPaths" : {
          "userId" : [ ],
          "status" : [ ]
        },
        "direction" : "forward",
        "indexBounds" : {
          "userId" : [
            "[\"59e022d33f239800129c61c7\", \"59e022d33f239800129c61c7\"]"
          ],
          "status" : [
            "[\"repayed\", \"repayed\"]"
          ]
        }
      }
    },
    "rejectedPlans" : [
      {
        "stage" : "FETCH",
        "filter" : {
          "status" : {
            "$eq" : "repayed"
          }
        },
        "inputStage" : {
          "stage" : "IXSCAN",
          "keyPattern" : {
            "userId" : 1
          },
          "indexName" : "userId_1",
          "multiKeyPaths" : {
            "userId" : [ ]
          },
          "direction" : "forward",
          "indexBounds" : {
            "userId" : [
              "[\"59e022d33f239800129c61c7\", \"59e022d33f239800129c61c7\"]"
            ]
          }
        }
      }
    ]
  },
  "ok" : 1
}

***命中索引， 跟 query 的各个字段顺序不相关，如我们猜测***

***有趣部分再来， 我们删掉索引{userId:1}***

> db.loans.dropIndex({"userId":1})
{ "nIndexesWas" : 3, "ok" : 1 }

> db.loans.find({"userId" : "59e022d33f239800129c61c7", }).explain()
{
  "queryPlanner" : {
    "namespace" : "cashLoan.loans",
    "parsedQuery" : {
      "userId" : {
        "$eq" : "59e022d33f239800129c61c7"
      }
    },
    "winningPlan" : {
      "stage" : "FETCH",
      "inputStage" : {
        "stage" : "IXSCAN",
        "keyPattern" : {
          "userId" : 1,
          "status" : 1
        },
        "indexName" : "userId_1_status_1",
        "multiKeyPaths" : {
          "userId" : [ ],
          "status" : [ ]
        },
        "direction" : "forward",
        "indexBounds" : {
          "userId" : [
            "[\"59e022d33f239800129c61c7\", \"59e022d33f239800129c61c7\"]"
          ],
          "status" : [
            "[MinKey, MaxKey]"
          ]
        }
      }
    },
    "rejectedPlans" : [ ]
  },
  "ok" : 1
}

***DB 执行分析器觉得索引{userId:1, status:1} 能更优***

***没有命中复合索引 ，这个是因为status 不是 leading field***

> db.loans.find({ "status" : "repayed" }).explain()
{
  "queryPlanner" : {
    "namespace" : "cashLoan.loans",
    "parsedQuery" : {
      "status" : {
        "$eq" : "repayed"
      }
    },
    "winningPlan" : {
      "stage" : "COLLSCAN",
      "filter" : {
        "status" : {
          "$eq" : "repayed"
        }
      },
      "direction" : "forward"
    },
    "rejectedPlans" : [ ]
  },
  "ok" : 1
}


***再换个角度sort 一遍， 与前面query & sort 互换 ，之前是***
> db.loans.find({userId:1}).sort({ "status" : "repayed" }) 
看看有啥不一样？


> db.loans.find({ "status" : "repayed" }).sort({userId:1}).explain()
{
  "queryPlanner" : {
    "namespace" : "cashLoan.loans",
    "parsedQuery" : {
      "status" : {
        "$eq" : "repayed"
      }
    },
    "winningPlan" : {
      "stage" : "FETCH",
      "filter" : {
        "status" : {
          "$eq" : "repayed"
        }
      },
      "inputStage" : {
        "stage" : "IXSCAN",
        "keyPattern" : {
          "userId" : 1,
          "status" : 1
        },
        "indexName" : "userId_1_status_1",
        "multiKeyPaths" : {
          "userId" : [ ],
          "status" : [ ]
        },
        "direction" : "forward",
        "indexBounds" : {
          "userId" : [
            "[MinKey, MaxKey]"
          ],
          "status" : [
            "[MinKey, MaxKey]"
          ]
        }
      }
    },
    "rejectedPlans" : [ ]
  },
  "ok" : 1
}

***如猜测，命中索引***

再来玩1玩，确认下***leading filed***试验：

> db.loans.dropIndex("userId_1_status_1")
{ "nIndexesWas" : 2, "ok" : 1 }

> db.loans.getIndexes()
[
  {
    "v" : 2,
    "key" : {
      "_id" : 1
    },
    "name" : "_id_",
    "ns" : "cashLoan.loans"
  }
]

> db.loans.createIndex({status:1, userId:1})
{
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 1,
  "numIndexesAfter" : 2,
  "ok" : 1
}

> db.loans.getIndexes()
[
  {
    "v" : 2,
    "key" : {
      "_id" : 1
    },
    "name" : "_id_",
    "ns" : "cashLoan.loans"
  },
  {
    "v" : 2,
    "key" : {
      "status" : 1,
      "userId" : 1
    },
    "name" : "status_1_userId_1",
    "ns" : "cashLoan.loans"
  }
]

> db.loans.find({ "status" : "repayed" }).explain()
{
  "queryPlanner" : {
    "namespace" : "cashLoan.loans",
    "parsedQuery" : {
      "status" : {
        "$eq" : "repayed"
      }
    },
    "winningPlan" : {
      "stage" : "FETCH",
      "inputStage" : {
        "stage" : "IXSCAN",
        "keyPattern" : {
          "status" : 1,
          "userId" : 1
        },
        "indexName" : "status_1_userId_1",
        "multiKeyPaths" : {
          "status" : [ ],
          "userId" : [ ]
        },
        "direction" : "forward",
        "indexBounds" : {
          "status" : [
            "[\"repayed\", \"repayed\"]"
          ],
          "userId" : [
            "[MinKey, MaxKey]"
          ]
        }
      }
    },
    "rejectedPlans" : [ ]
  },
  "ok" : 1
}

status_1_userId_1 有这个索引的前提，去查 leading fields -- status: xx 会中

> db.loans.getIndexes()
[
  {
    "v" : 2,
    "key" : {
      "_id" : 1
    },
    "name" : "_id_",
    "ns" : "cashLoan.loans"
  },
  {
    "v" : 2,
    "key" : {
      "status" : 1,
      "userId" : 1
    },
    "name" : "status_1_userId_1",
    "ns" : "cashLoan.loans"
  }
]

> db.loans.find({"userId" : "59e022d33f239800129c61c7", }).explain()
{
  "queryPlanner" : {
    "namespace" : "cashLoan.loans",
    "parsedQuery" : {
      "userId" : {
        "$eq" : "59e022d33f239800129c61c7"
      }
    },
    "winningPlan" : {
      "stage" : "COLLSCAN",
      "filter" : {
        "userId" : {
          "$eq" : "59e022d33f239800129c61c7"
        }
      },
      "direction" : "forward"
    },
    "rejectedPlans" : [ ]
  },
  "ok" : 1
}

status_1_userId_1 有这个索引的前提，去查 非leading fields -- user_id: xx 没中，全表扫描

所以 注意使用上 使用频率上 区分高的/常用的， 应该使用于混合索引，在前面作为leading fields， 

