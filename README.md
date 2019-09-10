### Static

``` javascript
  let _trans = await Trans.find({status:true}).populate(['userId', 'planId']);
  for (let item of _trans) {
      if (!item.days || !item.status || item.isOver) continue; 
      let ret = [0, 0, 0] 
      if (!item.planId.ones) ret[1] = new Bn(item.amount).multipliedBy(item.planId.bonus).dividedBy(100).toNumber();
      if (item.days === 1) { 
          if (item.planId.isReturn) 
            ret[0] = new Bn(ret[0]).plus(item.amount).toNumber(); 
          if (item.planId.ones > 0) 
            ret[2] = new Bn(ret[2]).plus(item.amount).multipliedBy(item.planId.ones).dividedBy(100).toNumber(); 
          item.isOver = true;
      }
      item.days--;
      await item.save();
      if (ret[0] > 0) await User.updateOne({_id: item.userId._id}, {
          '$inc': {daysAmount: ret[0]},
          '$push': {logs: {type: 3, amount: ret[0]}}
      });
      if (ret[2] > 0) await User.updateOne({_id: item.userId._id}, {
          '$inc': {daysAmount: ret[2]},
          '$push': {logs: {type: 4, amount: ret[2]}}
      });
      if (ret[1] > 0) await User.updateOne({_id: item.userId._id}, {
          '$inc': {
              staticAmount: ret[1],
              daysAmount: ret[1]
          }, '$push': {logs: {type: 1, amount: ret[1]}}
      });
  }
 ```
 
 ### Dynamic
 
 ``` javascript
   let returnTable = [
      [5000],
      [7000, 3500, 1750, 100, 100],
      [8000, 4000, 2000, 100, 100, 10, 10],
      [8000, 4000, 2000, 1000, 100, 100, 50, 10, 10],
      [9000, 4500, 2250, 1125, 100, 100, 50, 50, 10, 10],
      [9000, 4500, 2250, 1125, 200, 100, 100, 50, 10, 10, 10],
      [9000, 4500, 2250, 1125, 562.5, 100, 100, 50, 50, 50, 50, 50],
      [10000, 5000, 2500, 1250, 625, 312.5, 156, 50, 50, 50, 50, 50, 50, 50],
      [10000, 5000, 2500, 1250, 625, 312.5, 156, 100, 100, 100, 100, 100, 100, 100]
  ];
  let users = await User.find();
  for (const item of users) {
      //计算返利
      let invest = [];
      await fl(item, 0, returnTable[item.grade].length, returnTable[item.grade], invest,item);
      let a = 0;
      for(const de of invest){ a = new Bn(a).plus(de).toNumber() };
      if (a > 0) await User.updateOne({_id: item._id}, {
          '$inc': {daysAmount: parseFloat(a)},
          '$push': {logs: {type: 2, amount: a}}
      })
  }
 ```
