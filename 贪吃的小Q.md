## 问题描述
![img](https://uploadfiles.nowcoder.com/images/20180405/8618764_1522929378680_2325F0555B49FF8566C5A0F3EA0822E9)  
牛客上看到的题，没有跑过测试用例，不保证正确。  

## 解
```JS
function a(food, day) {
  let firstDay = food;

  for(;firstDay > 0;firstDay --){
    if(inner(firstDay, food - firstDay, day - 1)) {
      break;
    }
  }
  return firstDay;

  function inner(lastDay, foods, days) {
    if(days === 0) {
      if(foods >= 0)
        return true;
      return false;
    }

    if(lastDay / 2 > foods) {
      return false;
    }

    if(foods < days) {
      return false;
    }

    lastDay = Math.ceil(lastDay / 2);
    foods -= lastDay;
    days --;
    return inner(lastDay, foods, days);
  }
}

a(7, 3); // 4
```
