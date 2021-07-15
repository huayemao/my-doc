

## 广度优先搜索 BFS

### [104.二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)


广度优先搜索的队列里存放的是「当前层的所有节点」。每次拓展下一层的时候，不同于广度优先搜索的每次只从队列里拿出一个节点，需要将队列里的所有节点都拿出来进行拓展，保证每次拓展完的时候队列里存放的是当前层的所有节点。


``` js
/**
 * @param {TreeNode} root
 * @return {number}
 */
var maxDepth = function (root) {
    if (!root) return 0;
    let node, depth = 0;
    const queue = [root];

    while (queue.length > 0) {
        let rowSize = queue.length;
        while (rowSize > 0) {
            node = queue.shift()
            node.left && queue.push(node.left);
            node.right && queue.push(node.right);
            rowSize--
        }
        depth++;
    }
    return depth
};
```

### [112.路径总和](https://leetcode-cn.com/problems/path-sum/submissions/)

``` js
/**
 * @param {TreeNode} root
 * @param {number} targetSum
 * @return {boolean}
 */
var hasPathSum = function (root, targetSum) {
    if(!root) return false
    const queue = [{ node: root, sum: root.val }];
    const childrenKeys = ['left', 'right'];

    function enQueue(key, node, sum) {
        if (node[key]) {
            const newSum = sum + node[key].val;
            queue.push({
                node: node[key],
                sum: newSum
            })
        }
    }

    while (queue.length !== 0) {
        const { node, sum } = queue.shift();

        // 叶子节点则判断是否满足条件，或者进行下一趟
        if (childrenKeys.every(key => !node[key])) {
            if (sum === targetSum) return true;
            continue
        }
        childrenKeys.forEach(key => enQueue(key, node, sum))
    };

    return false


}
```


### [200.岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)


常犯错误： grid[y][x] x和y搞混带来的数组越界问题


``` js
/**
 * @param {character[][]} grid
 * @return {number}
 */
var numIslands = function (grid) {
    let res = 0;
    const rowCount = grid.length;
    const colCount = grid[0].length;
    if (grid.length === 0) {
        return 0
    }

    for (let i = 0; i < rowCount; i++) {
        for (let j = 0; j < colCount; j++) {
            const el = grid[i][j];
            if (el === '1') {
                res += 1;
                bfs(j, i);
            }
        }
    }

    return res

    function isRangeValid(x, y) {
        return x >= 0 && y >= 0 && x < colCount && y < rowCount
    }

    function bfs(x, y) {
        grid[y][x] = '0';
        const queue = [[x, y]];
        while (queue.length !== 0) {
            for (let size = queue.length; size > 0; size--) {
                const [x, y] = queue.shift();
                const arr = [
                    [x - 1, y],
                    [x, y - 1],
                    [x, y + 1],
                    [x + 1, y],
                ];

                arr.forEach(([x, y]) => {
                    if (isRangeValid(x, y) && grid[y][x] === '1') {
                        grid[y][x] = '0';
                        queue.push([x, y])
                    }
                });
            }
        }
    }
};
```

## 深度优先搜索 DFS

### [200.岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)

``` js
/**
 * @param {character[][]} grid
 * @return {number}
 */
var numIslands = function (grid) {
    let res = 0;
    const rowCount = grid.length;
    const colCount = grid[0].length;
    if (grid.length === 0) {
        return 0
    }

    for (let i = 0; i < rowCount; i++) {
        for (let j = 0; j < colCount; j++) {
            const el = grid[i][j];
            if (el === '1') {
                res += 1;
                dfs(j, i);
            }
        }
    }

    return res

    function isRangeValid(x, y) {
        return x >= 0 && y >= 0 && x < colCount && y < rowCount
    }

    function dfs(x, y) {
        const arr = [
            [x - 1, y],
            [x, y - 1],
            [x, y + 1],
            [x + 1, y],
        ];

        arr.forEach(([x, y]) => {
            if (isRangeValid(x, y) && grid[y][x] === '1') {
                grid[y][x] = '0';
                dfs(x, y);
            }
        })

    }
};
```


