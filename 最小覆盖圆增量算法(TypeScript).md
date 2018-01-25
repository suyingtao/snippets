时间复杂度O(n)

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