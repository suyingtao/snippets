### 2 Sum问题
```JS
/* 
 * 方法1：遍历数组构造hash，再遍历一次数组，只要hash中存在sum - num则有解。
 * 方法2: 排序，然后头尾指针不断逼近寻找解。
 */
function find2sum (arr, sum) {
    const r = [];
    const hash = {};
    for(let i in arr) {
        hash[arr[i]] = i + 1;
    }
    
    for(let i in arr) {
        if(hash[sum - arr[i]] && hash[sum - arr[i]] !== i + 1 && hash[sum - arr[i]] > i) {
            r.push([arr[i], sum - arr[i]]);
        }
    }
    
    return r;
}
```
### k sum

    
```JS
/* 
 * 当n = 2时可以用2Sum的方法，略微降低时间复杂度。
 */
function KSum(arr, n, sum, temp, result){
    if(!result) result = [];

    if(sum === 0 && n === 0) {
        // 这里需要深拷贝temp
        result.push(temp.concat());

        return ;
    }
    
    if(n <= 0) {
        return ;
    }
    
    for(let i = 0; i < arr.length; i++) {
        temp.push(arr[i]);

        KSum(arr.slice(i + 1, arr.length), n - 1, sum - arr[i], temp, result);

        temp.pop();
    }
    
    return result;
}

```