## 基本操作
1. 倒叙查询：
db.getCollection('didi_request_log').find({"parameters.merchantID":"6110200182"}).sort({'nowTime':-1})
2. 模糊查询
db.getCollection('didi_request_lead_log').find({"nowTime": {$regex: '2018-02-03', $options:'i'}}).sort({'nowTime':-1})
