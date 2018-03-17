### 描述

求出覆盖n个点的最小圆

### 增量法  

假设圆O是前i-1个点得最小覆盖圆，加入第i个点，如果在圆内或边上则什么也不做。否则,**新得到的最小覆盖圆肯定经过第i个点**。
然后以第i个点为基础（半径为0），重复以上过程依次加入第j个点，若第j个点在圆外，则最小覆盖圆必经过第j个点。
重复以上步骤（因为最多需要三个点来确定这个最小覆盖圆，所以重复三次）。遍历完所有点之后，所得到的圆就是覆盖所有点得最小圆。  
### 证明  

最小圆必定是可以通过不断放大半径，直到所有以任意点为圆心，半径为半径的圆存在交点，此时的半径就是最小圆。所以上述定理可以通过这个思想得到。这个做法复杂度是O(n)的，当加入圆的顺序随机时，因为三点定一圆，所以不在圆内概率是3/i,求出期望可得是O(n)。

### TypeScript代码实现
```typeScript
interface Position {
    x: number;
    y: number;
}

interface Circle {
    center: Position;
    radius: number;
}

/**
 * @desc 求两点间距离
 * @param a: Position
 * @param b: Position
 */
function len(a: Position, b: Position) {
    return Math.sqrt(Math.pow(a.x - b.x, 2) + Math.pow(a.y - b.y, 2));
}

/**
 * @desc 求三角形外接圆
 * @param a
 * @param b
 * @param c
 */
function circleOfTriangle(a: Position, b: Position, c: Position): Circle {

    const a1 = 2 * (b.x - a.x),
          b1 = 2 * (b.y - a.y),
          c1 = b.x * b.x + b.y * b.y - a.x * a.x - a.y * a.y,
          a2 = 2 * (c.x - b.x),
          b2 = 2 * (c.y - b.y),
          c2 = c.x * c.x + c.y * c.y - b.x * b.x - b.y * b.y;

    const center = {
        x: (c1 * b2 - c2 * b1) / (a1 * b2 - a2 * b1),
        y: (a1 * c2 - a2 * c1) / (a1 * b2 - a2 * b1)
    };

    const circle = {
        center: center,
        radius: len(a, center)
    };

    return circle;
}

/**
 * @desc 最小覆盖圆
 */
function minCircle(pArr: Array<Position>) {
    let tempCircle = {
        center: pArr[0],
        radius: 0
    };

    for (let i = 0; i < pArr.length; i++) {
        if (len(pArr[i], tempCircle.center) > tempCircle.radius) {
            tempCircle.center = pArr[i];
            tempCircle.radius = 0;

            for (let j = 0; j < i; j++) {
                if (len(pArr[j], tempCircle.center) > tempCircle.radius) {
                    tempCircle.center = {
                        x: (pArr[i].x + pArr[j].x) / 2,
                        y: (pArr[i].y + pArr[j].y) / 2
                    };
                    tempCircle.radius = len(pArr[i], pArr[j]) / 2;

                    for (let k = 0; k < j; k++) {
                        if (len(pArr[k], tempCircle.center) > tempCircle.radius) {
                            tempCircle = circleOfTriangle(pArr[i], pArr[j], pArr[k]);
                        }
                    }
                }
            }
        }
    }

    return tempCircle;
}


```
