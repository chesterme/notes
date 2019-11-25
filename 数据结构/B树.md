## 基本概念
1. 一棵M叉查找树可以有M路分支，随着分支增加，树的深度在减少。一棵完全二叉树的高度大约为：$log_2^N$，而一棵完全M叉树的高度大约为：$log_m^N$
2. 每一个磁盘IO请求指定了要访问的磁盘地址，即块号。一个块是一个逻辑单元，它包含固定数目的连续扇区。一个块的大小通常在[512byte, mkb]。数据在磁盘和主存储器之间以块为单位进行传输。
3. 在一个顺序访问模式中，连续的磁盘IO请求会请求处于相同的磁道或者相邻的磁道上的块。第一个块可能需要一次磁盘寻道，但是相继的IO请求既不需要磁盘寻道，也不需要对相邻磁道的寻道。
4. 在一个随机访问模式中，相继的IO请求会请求那些随机位于磁盘上的块，每一个IO请求都需要一次磁盘寻道。

## B树的特征
1. 是一棵M叉树
2. 数据项存储在树叶上，所有的树叶都在相同的高度上
3. 非叶子节点(根节点除外)上存储了M-1个关键字，每一个关键字表示一个范围，它有一个对应的指针域，指向属于该范围的下一个非叶子节点或者叶子节点
4. 除了根节点外，所有非叶子节点的儿子节点数在[M/2, M)之间，确保每个非叶子节点存储的内容不会太少
5. 所有树叶在相同的深度上都有[L/2, L)之间的数据项，树叶半满将确保B数不会退化成为一棵简单的二叉树
6. 一棵M叉树的非叶子节点结构如下，其中porint_x是一个指针域，它指向一个叶子节点或下一个非叶子节点，它们的值都小于关键字key_x的值，而key_x表示一个关键字，所有的关键字组成一个有序的线性表，非降序排列。
```text
    ---------------------------------------------------------------------------
    | point_1 | key_1 | point_2 | key_2 | ... | point_m-1 | key_m-1 | point_m |
    ---------------------------------------------------------------------------
```
7. 一棵M叉树的叶子节点结构如下,大小为[L/2, L]，其中L表示在一个块中最多可以保存多少条记录
```text
    ------------
    | record_1 |
    | record_2 |
    | ...      |
    | record_L |
    ------------
```

## 一个例子
1. 假设有1千万条记录需要存储，每一条记录的大小为256byte，每一个关键字的大小为32byte
2. 假设所有记录都不能保存在内存中，在磁盘中保存，一个块的大小为8192byte
3. 假设使用随机访问的模式，即每一个块都需要一次磁盘寻道来读取。
4. 使用M叉树来保存记录，每一个非叶子节点(根节点除外)都有[(M-1)/2, M-1]个关键字，每个关键字都按照从小到大有序排列，有[(M-1)/2 + 1, M]个指针。
5. 指针point_x指向磁盘中的一个块号block_x，在这个块中存储着对应的记录，这些记录在关键字上满足：$\forall key \in [key_{x-1}, key_x)$。
6. 一个非叶子节点的所有关键字所占空间大小为：$32*(M - 1)=32M-32$，假设一个指针域大小为4byte，则非叶子节点的大小为：$32M-32+4M=36M-32$，在一个块中最多可以容纳关键字的数量为：$8192 = 36M - 32$, 得到M = 228，即一个非叶子节点的分支数量为：[114, 228]。
7. 在一个块中最多可容纳记录的数量为：$8192 = 256x$, 得到x为32，若用一个叶子节点表示一个块，那么一个叶子节点所保存的记录数量为：[16, 32]。
8. 假设一个叶子节点只使用一半的容量，即保存16条记录，那么1千万条记录大概需要：$16x = 10000000$，得到x为625000个叶子节点，即需要625000个块
9. 假设一个非叶子节点只使用一半的容量，即存储114个关键字，那么从根节点开始往下搜索，直到叶子节点，大概需要磁盘IO次数为: $log_{m/2}^{N} = log_{114}^{10000000} \approx  4$


## 结点结构
1. 一个有序的关键字列表
2. 一个与关键字对应的指针列表
3. 一个标识是否是叶子结点的标志

```javascript
// B树结点结构
function TreeNode(keyList, pointList, dataList, isLeaf){
    this.keyList = keyList;     // 有序的关键字列表
    this.pointList = pointList; // 对应关键字的指针列表
    this.isLeaf = isLeaf; // 标识该节点是否是叶子结点
}

// B树
function BTree(root, branchNumber){
    this.root = root; // B树的根节点
    this.branchNumber = branchNumber; // 分支数
}
```

## 查找一个元素
假设需要查找关键字为v的所有记录：
1. 以根节点为当前结点
2. 检查当前结点，即遍历关键字列表，查找最小的i使得关键字$k_i >= v$
3. 如果$k_i == v$，那么将当前结点置为由$point_{i+1}$指向的结点
4. 如果$k_i > v$，那么将当前结点置为由$point_{i}$指向的结点
5. 如果没有找到这样的$k_i$，则说明$v >= k_{m-1}$，则将当前结点置为$point_m$指向的结点
6. 重复步骤2，直到访问到叶子结点为止

伪代码表示为：
```pseudocode
// x表示B树中的一个结点，k表示需要查找的关键字
B-TREE-SEARCH(x, k){
    i = 1;
    // 找出最小的i，使得x.key_i >= k
    while(i <= x.n && x.key_i < k){
        i++;
    }
    // 如果在当前结点的关键字列表中找到k，则直接返回(x, i)
    if(i <= x.n && x.key_i == k){
        return (x, i);
    }
    // 如果已经来到B树的叶结点，则表示B树中不存在该元素
    if(x.isLeaf){
        return null;
    }
    // 如果未来到B树的叶结点，则需要继续往下搜索
    else{
        B-TREE-SEARCH(x.point_i, k);
    }
}
```

javascript的递归实现：
```javascript
// 在b树中查找一个元素
// subRoot表示一棵子树的根节点，key表示需要查找的元素
BTree.prototype.search = function(subRoot, key){
    var i = 1;
    // 从当前结点的关键字序列中找到一个最小的i，使得subRoot.key_i >= key
    while(i <= subRoot.keyNumber && subRoot.keyList[i] < key){
        i++;
    }
    // 如果找到了，则直接返回
    if(i <= subRoot.keyNumber && subRoot.keyList[i] == key){
        return new PairData(subRoot, i);
    }
    // 如果已经来到了树叶结点，说明B树中不存在该元素
    if(subRoot.isLeaf){
        console.log("B树中不存在该元素:" + key)
        return null;
    }
    else{
        this.search(subRoot.pointList[i], key);
    }
}
```

javascript的非递归实现：
```javascript
// 查找一个元素
BTree.prototype.find = function(key){
    if(this.isEmpty()){
        console.log("b树不能为空");
        return null;
    }
    // 设置当前结点为根节点
    var currentNode = this.root;
    var keyList = null;
    while(!currentNode.isLeaf){
        // 遍历关键字列表，找到最小的i，使得key_i >= key
        keyList = currentNode.keyList;
        var counter = 1;
        var listNode = keyList.header.next;
        while(listNode != keyList.tail){
            // 如果key_i == key，则将当前结点设置为pointList[i + 1]
            if(listNode.data == key){
                currentNode = pointList.getKth(counter + 1);
                break;
            }
            // 如果key_i > key, 则将当前结点设置为pointList[i]
            else if(keyList.data> key){
                currentNode = pointList.getKth(counter);
                break;
            }
            else{
                listNode = listNode.next;
                counter++;
            }
        }

        // 如果key >= key_m-1，则将当前结点设置为pointList[m]
        if(listNode == keyList.tail){
            currentNode = pointList.getKth(counter);
        }

        if(currentNode == null){
            console.log("不存在该元素: " + key);
            return null;
        }
    }
    
    // 遍历关键字列表
    var keyList = currentNode.keyList;
    var pointList = currentNode.pointList;
    var counter = 1;
    for(var listNode = keyList.header.next; listNode != keyList.tail; listNode = listNode.next){
        if(keyList[i] == key){
            return pointList.getKth(counter);
        }
        counter++;
    }
    console.log("不存在该元素: " + key);
    return null;
}
```

## 打印关键字key对应的所有元素
```javascript
// 打印关键字为key的所有记录
BTree.prototype.printAll = function(key){
    // 从B树中查找关键字对应的记录
    var dataNode = this.find(key);
    if(dataNode == null){
        console.log("不存在该元素：" + key);
        return null;
    }

    while(dataNode != null){
        console.log(dataNode.data);
        dataNode = dataNode.next;
    }
}
```

## 创建一个空的B树
伪代码：
```pseudocode
B-TREE-CREATE(T):
    // 创建一个B树的空结点
    x = ALLOCATE-NODE(); 
    x.isLeaf = true;
    // 初始化关键字数量为0
    x.n = 0;
    // 使它成为B树的根节点
    T.root = x;
```

## 分裂B树中的结点
伪代码:
```pseudocode
// x表示关键字序列已满的结点y的父结点，i表示父结点x中指针序列的下标，它使得x.point_i指向结点y
B-TREE-SPLIT-CHILD(x, i):
    // 新建一个空白的结点
    z = ALLOCATE-NODE();
    // 使用y来表示需要分裂的结点
    y = x.point_i;
    // 将y结点的一半数据复制到结点z中
    z.isLeaf = y.isLeaf;
    // t表示最大分支数的一半
    z.n = t - 1;
    // 复制关键字
    for j = 1 to t-1:
        z.key_j = y.key_(t+j);
    // 如果是非叶子结点，还需要复制指针域
    if not y.isLeaf:
        for j = 1 to t-1:
            z.point_j = y.point_(t+j);
    // 将新节点z插入到结点y的父结点中
    for j = x.n + 1 downto i + 1:
        x.point_(j + 1) = x.point_j;
    x.point_(i + 1) = z;
    // 将新结点z和旧结点y的中间元素放置在它们的父结点x上
    for j = x.n downto i:
        x.key_j = x.key_(j+1);
    x.key_i = y.key_t;
    // 从结点y中删除关键字key_t
    y.key_t = null;
    // 更新y结点的关键字数量
    y.n = t - 1;
    // 更新父结点x的关键字数量
    x.n = x.n + 1;
```

javascript描述：
```javascript
// 在b树中分裂一个关键字序列已满的结点
// parentNode表示已满结点的父结点，i表示父结点指针序列的下标，parentNode.point_i指向已满结点
BTree.prototype.splitChild = function(parentNode, i){
    // 需要分裂的结点
    var currentNode = parentNode.pointList[i];
    // t表示最小分支数
    var t = this.minBranchNumber;
    // 建立一个新结点
    var newNode = new TreeNode(new Array(), new Array(), t-1, currentNode.isLeaf);
    newNode.keyList[0] = newNode.pointList[0] = null;
    // 复制关键字到新节点上
    for(var j = 1; j <= t - 1; j++){
        newNode.keyList[j] = currentNode.keyList[t + j];
    }
    // 删除已经复制的元素
    for(var j = t - 1; j >= 1; j--){
        currentNode.keyList.pop();
    }

    // 如果是非叶子结点，则需要复制指针域
    if(!currentNode.isLeaf){
        for(var j = 1; j <= t; j++){
            newNode.pointList[j] = currentNode.pointList[t + 1 + j];
        }
        // 删除已经复制的元素
        for(var j = t; j >= 1; j--){
            currentNode.pointList.pop();
        }
    }
    // 将新节点插入到当前结点的父结点上
    for(var j = parentNode.keyNumber + 1; j >= i + 1; j--){
        parentNode.pointList[j + 1] = parentNode.pointList[j];
    }
    parentNode.pointList[j + 1] = newNode;

    // 将关键字插入到父结点上
    var midKey = currentNode.keyList[t];
    currentNode.keyList.pop();
    currentNode.keyNumber = t - 1;

    for(var j = parentNode.keyNumber; j >= i; j--){
        parentNode.keyList[j + 1] = parentNode.keyList[j];
    }
    parentNode.keyList[j + 1] = midKey;
    parentNode.keyNumber++;
}
```

## 往B树中插入关键字
伪代码：
```pseudocode
// T表示B树，k表示需要插入的关键字
B-TREE-INSERT(T, k):
    // 从B树的根节点开始往下搜索待插入的位置
    r = T.root;
    // 如果根节点的关键字序列已经满，需要进行分裂
    if r.n == 2t - 1:
        // 新建一个空白的结点，让它成为根节点r的父结点，即成为B树的根节点
        s = ALLOCATE-NODE();
        s.isLeaf = false;
        s.n = 0;
        s.point_1 = r;
        T.root = s;
        // 分裂已满的结点
        B-TREE-SPLIT-CHILD(s, 1);
        // 往已经完成分裂的树中重新插入关键字k
        B-TREE-INSRET-NONFULL(s, k);
    // 如果根节点的关键字序列未满，则直接插入在对应位置
    else:
        B-TREE-INSERT-NOFULL(r, k);
```

javascript描述：
```javascript
// 插入一个元素
BTree.prototype.insert = function(key){
    // 从B树的根节点开始往下搜索关键字的位置
    var rootNode = this.root;
    // 如果根节点的关键字序列已满，则需要分裂成两个子结点，然后重置一个根节点
    if(rootNode.keyNumber == this.branchNumber - 1){
        // 新建一个结点作为B树的根节点
        var newNode = new TreeNode(new Array(), new Array(), 0, false);
        newNode.keyList[0] = newNode.pointList[0] = null;
        newNode.pointList[1] = rootNode;
        this.root = newNode;
        // 分裂已满的结点
        this.splitChild(newNode, 1);
        // 往已完成分裂的B树中插入关键字key
        this.insertNonFull(newNode, key);
    }
    else{
        this.insertNonFull(rootNode, key);
    }
}
```

伪代码描述：
```pseudocode
// x表示B树中的一个子结点， k表示待插入的关键字
B-TREE-INSERT-NONFULL(x, k):
    i = x.n;
    // 如果是叶子结点，那么就不需要处理指针域，即它的指针域为空
    if x.isLeaf:
        // 结点x的关键字序列中，找到一个最小的i，使得x.key_i >= k
        while i >= 1 && k < x.key_i:
            x.key_(i + 1) = x.key_i;
            i--;
        x.key_(i+1) = k;
        x.n = x.n + 1;
    else:
        while i >= 1 && k < x.key_i:
            i--;
        i++;
        // 从磁盘中读取x.point_i指向的对象
        DISK-READ(x.point_i);
        // 如果该对象的关键字序列已满，则需要分裂
        if x.point_i.n == 2t - 1:
            B-TREE-SPLIT-CHILD(x, i);
            if k > x.key_i:
                i++;
        B-TREE-INSERT-NONFULL(x.point_i, k);     
```

javascript描述：
```javascript
// 往子树中插入一个关键字
BTree.prototype.insertNonFull = function(subRoot, key){
    var i = subRoot.keyNumber;
    // 如果是叶子结点，则不需要处理指针域
    if(subRoot.isLeaf){
        // 插入到关键字序列
        while(i >= 1 && subRoot.keyList[i] > key){
            subRoot.keyList[i + 1] = subRoot.keyList[i]
            i--;
        }
        subRoot.keyList[i + 1] = key;
        subRoot.keyNumber++;
    }
    else{
        // 找到下一个子结点
        while(i >= 1 && subRoot.keyList[i] > key){
            i--;
        }
        i++;
        // 如果已满，则需要分裂
        if(subRoot.pointList[i].keyNumber == this.branchNumber - 1){
            this.splitChild(subRoot, i);
            if(subRoot.keyList[i] < key){
                i++;
            }
        }
        this.insertNonFull(subRoot.pointList[i], key);
    }
}
```

## 删除一个元素
伪代码
```pesudocode
// x表示一棵子树的根节点，k表示需要删除的关键字
B-TREE-DELETE(x, k):
    // 从当前结点的关键字序列中查找k
    isFound = false;
    for i = 1 to x.n:
        if(x.key[i] == k)
            isFound = true
            break;
    // 如果k在当前结点中
    if isFound == true:
        // 如果当前结点时叶子结点，则直接删除
        if x.isLeaf == true:
            for j = i to x.n - 1:
                x.key[j] = x.key[j + 1];
            x.n--;
        // 如果是内部结点
        else:
            // 关键字k的紧邻前驱，包含小于k的关键字元素
            y = x.point[i];
            // 关键字k的紧邻后继，包含大于k的关键字元素
            z = x.point[i + 1];
            // 如果k的前驱至少有t个关键字，则将它最大的一个元素上移，替换要删除的关键字k
            if y.n >= t:
                max = B-TREE-FINDMAX(y);
                B-TREE-DELETE(y, max);
                s.key[i] = max;
            // 如果k的后继至少包含t个关键字，则将它最小的一个元素上移，替换掉要删除的关键字k
            else if z.n >= t:
                min = B-TREE-FINDMIN(z);
                B-TREE-DELETE(z, min);
                s.key[i] = min;
            // 如果k的前驱和后继都只有t-1个关键字，则将前驱、k和后继合并成一个结点，然后在新节点中删除k
            else if y.n == z.n == t-1:
                // 合并
                j = i + 1;
                y.key[j] = k;
                for m = 1 to z.n:
                    y.key[++j] = z.key[m];
                y.n = y.n + 1 + z.n;

                // 从当前结点中删除k
                for j = i to x.n - 1:
                    x.key[j] = x.key[j + 1];
                for j = i + 1 to x.n:
                    x.point[j] = x.point[j + 1];
                x.n--;

                B-TREE-DELETE(y, k);

    // k不在当前结点中
    else:
        // 找到一个最小的i，使得x.key[i] > k
        for i = 1 to x.n:
            if x.key[i] > k:
                break;

        // 下一个查找的结点
        target = x.point[i];
        // 如果它只有t-1个关键字，若直接删除，则不符合约束，因此它需要向相邻结点借一个关键字，
        if target.n == t-1:
            // 目标结点的左兄弟
            y = x.point[i -1];
            // 如果左兄弟的关键字数量至少为t，则将它的最大关键字上移至父结点，将父结点的key[i]下移至target结点，然后从target结点中删除k
            if(y.n >= t){
                max = B-TREE-FINDMAX(y);
                B-TREE-DELETE(y, max);
                
                // 将父结点的关键字key[i - 1]下降到它的子结点中
                for j = target.n downto i:
                    target.key[j + 1] = target.key[j];
                target.key[j + 1] = x.key[i - 1];
                target.n++;

                // 更新父结点关键字key[i-1]
                x.key[i-1] = max;
                B-TREE-DELETE(target, k);
            }

            // 目标结点的右兄弟
            z = x.point[i + 1];
            // 如果右兄弟的关键字数量至少为t，则将它的最小关键字上移至父结点，将父结点的key[i]下移至target结点，然后从target结点中删除k
            if(z.n >= t){
                min = B-TREE-FINDMIN(z);
                B-TREE-DELETE(z, min);

                // 将父结点的关键字key[i]下降到它的子结点中
                target.key[target.n + 1] = x.key[i];
                target.n++;

                // 更新父结点关键字key[i]
                x.key[i] = min;
                B-TREE-DELETE(target, k);
            }

            // 如果目标结点左右兄弟的关键字数量都为t-1，则合并目标结点和左兄弟，然后从新节点中删除k
            if(y.n == t-1 && z.n == t-1){
                j = y.n + 1;
                y.key[j] = x.key[i];
                for m = 1 to z.n:
                    y.key[++j] = z.key[m];
                y.n = y.n + 1 + z.n;

                // 从父结点中删除key[i]
                for j = i to x.n - 1:
                    x.key[j] = x.key[j + 1];
                for j = i to x.n:
                    x.point[j] = x.point[j + 1];
                x.n--;

                if(x.n == 0){
                    T.root = x.point[1];
                }

                B-TREE-DELETE(y, k);
            }
```

JavaScript描述：
```javascript
// 从一棵子树中删除一个元素
BTree.prototype.delete = function(subRoot, k){
    if(subRoot == null){
        console.log("子树不能为空");
        return false;
    }

    // t表示B树可容纳最小分支数
    var t = this.minBranchNumber;
    // 遍历当前结点
    var isFound = false;
    var i = null;
    for(i = 1; i <= subRoot.keyNumber; i++){
        if(subRoot.keyList[i] == k){
            isFound = true;
            break;
        }
    }

    // 如果在当前结点内找到k
    if(isFound){
        // 如果是叶子结点，则直接从关键字序列中删除k
        if(subRoot.isLeaf){
            for(var j = i; j < subRoot.keyNumber; j++){
                subRoot.keyList[j] = subRoot.keyList[j + 1];
            }
            subRoot.keyList.pop();
            subRoot.keyNumber--;

            return ;
        }

        // 如果是内部结点
        else{
            // y为关键字k的紧邻前驱，包含小于k的关键字元素
            var y = subRoot.pointList[i];
            // z为关键字k的紧邻后继，包含大于k的关键字元素
            var z = subRoot.pointList[i + 1];
            // 如果k的前驱至少有t个关键字，则将它最大的一个元素上移，替换要删除的关键字k
            if(y.keyNumber >= t){
                var max = this.findMax(y);
                this.delete(y, max);
                subRoot.keyList[i] = max;
            }
            // 如果k的后继至少有t个关键字，则将它最小的一个元素上移，替换要删除的关键字k
            else if(z.keyNumber >= t){
                var min = this.findMin(z);
                this.delete(z, min);
                subRoot.keyList[i] = min;
            }
            // 如果k的前驱和后继都只有t-1个关键字，则将前驱j、关键字k和后继z合并成一个结点，然后在新节点中删除k
            else if(z.keyNumber == y.keyNumber && z.keyNumber == t-1){
                // 合并关键字k
                var j = y.keyNumber + 1;
                y.keyList[j++] = k;
                // 合并后继结点z中的关键字
                for(var m = 1; m <= z.keyNumber; m++){
                    y.keyList[j++] = z.keyList[m];
                }
                y.keyNumber += (1 + z.keyNumber);

                // 从当前结点中删除k，即删除subRoot.keyList[i]
                for(var j = i; j < subRoot.keyNumber; j++){
                    subRoot.keyList[j] = subRoot.keyList[j + 1];
                }
                subRoot.keyList.pop();

                // 删除关键字k的后继结点，即删除subRoot.pointList[i + 1]
                for(var j = i + 1; j <= subRoot.keyNumber; j++){
                    subRoot.pointList[j] = subRoot.pointList[j + 1];
                }
                subRoot.pointList.pop();

                // 更新关键字数目
                subRoot.keyNumber--;

                // 如果当前节点的关键字列表为空，则以它的左儿子作为根节点
                // if(subRoot.keyNumber == 0){
                //     this.root = subRoot.pointList[1];
                // }

                // 从合并的结点中删除k
                return this.delete(y, k);
            }
        }
    }
    // 如果在当前结点中没有找到k，即k可能在它的子结点中
    else{
        // 找出对应的子结点
        var m = null;
        for(m = 1; m <= subRoot.keyNumber; m++){
            if(subRoot.keyList[m] > k){
                break;
            }
        }

        // 对应的子结点
        var target = subRoot.pointList[m];
         // 它的左兄弟
         var y = subRoot.pointList[m - 1];
         // 它的右兄弟
         var z = subRoot.pointList[m + 1];
        // 如果它只有t-1个关键字，即只含最小数量的关键字，若直接删除，则不符合约束，因此它需要向相邻结点借一个关键字，
        if(target.keyNumber == t-1){
            // 如果左兄弟的关键字数量至少为t，即它可以借一个关键字给当前结点
            // 可以将它的最大关键字上移至父结点，将父结点的key[m - 1]下移至target结点，然后从target结点中删除k
            if(y != null && y.keyNumber >= t){
                var max = y.keyList.pop();  
                // 将父结点的关键字key[i - 1]下移至target结点上
                var j = null;
                for(j = target.keyNumber; j >= 1; j--){
                    target.keyList[j + 1] = target.keyList[j];
                }
                target.keyList[1] = subRoot.keyList[m -1];
                subRoot.keyList[m - 1] = max;

                // 如果y是内部结点，还需要移动指针序列
                if(!y.isLeaf){
                    var maxPoint = y.keyList.pop();
                    for(j = target.keyNumber + 1; j >= 1; j--){
                        target.pointList[j + 1] = target.pointList[j]; 
                    }
                    target.pointList[1] = maxPoint;
                }
                y.keyNumber--;

                target.keyNumber++;
                return this.delete(target, k);
            }
            // 如果右兄弟的关键字数量至少为t，即它可以借一个关键字给当前结点
            // 将它的最小关键字上移至父结点，将父结点的key[m]下移至target结点，然后从target结点中删除k
            else if(z != null && z.keyNumber >= t){
                var min = z.keyList[1];
                var j = null;
                for(j = 1; j < z.keyNumber; j++){
                    z.keyList[j] = z.keyList[j + 1];
                }
                z.keyList.pop();

                // 将父结点的关键字key[m]下移至target结点上
                target.keyList.push(subRoot.keyList[m]);
                subRoot.keyList[m] = min;

                // 如果z是内部结点，还需要移动指针
                if(!z.isLeaf){
                    var minPoint = z.pointList[1];
                    for(j = 1; j <= z.keyNumber; j++){
                        z.pointList[j] = z.pointList[j + 1];
                    }
                    z.pointList.pop();
                    target.pointList.push(minPoint);
                }
                z.keyNumber--;

                target.keyNumber++;
                return this.delete(target, k);
            }
            // 如果相邻结点的关键字数量都是t-1，则将它与其中一个相邻结点合并，然后从新节点中删除k
            // 子结点只有一个右兄弟结点
            if(m == 1){
                // 合并target结点、父结点的keyList[m]、z结点
                if(z != null && z.keyNumber == t - 1){
                    // 合并
                    target.keyList.push(subRoot.keyList[m]);
                    for(var j = 1; j <= z.keyNumber; j++){
                        target.keyList.push(z.keyList[j]);
                    }

                    if(!z.isLeaf){
                        for(var j = 1; j <= z.keyNumber + 1; j++){
                            target.pointList.push(z.pointList[j]);
                        }
                    }
                    target.keyNumber += (1 + z.keyNumber);

                    // 从父结点中删除关键字keyList[m]
                    for(var j = m; j < subRoot.keyNumber; j++){
                        subRoot.keyList[j] = subRoot.keyList[j + 1];
                    }
                    subRoot.keyList.pop();

                    // 从父结点中删除结点z
                    for(var j = m + 1; j <= subRoot.keyNumber; j++){
                        subRoot.pointList[j] = subRoot.pointList[j + 1];
                    }
                    subRoot.pointList.pop();
                    subRoot.keyNumber--;

                    if(subRoot.keyNumber == 0){
                        this.root = subRoot.pointList[1];
                        subRoot = null;
                    }

                    // 从合并后的结点中删除k
                    return this.delete(target, k);
                }
            }
            // 如果子结点在最右侧
            else if(m == subRoot.keyNumber + 1){
                // 合并y、父节点的keyList[m]、target结点
                if(y != null && y.keyNumber == t - 1){
                    // 合并
                    y.keyList.push(subRoot.keyList[m - 1]);
                    for(var j = 1; j <= target.keyNumber; j++){
                        y.keyList.push(target.keyList[j]);
                    }

                    if(!target.isLeaf){
                        for(var j = 1; j <= target.keyNumber + 1; j++){
                            y.pointList.push(target.pointList[j]);
                        }
                    }
                    y.keyNumber += (1 + target.keyNumber);

                    // 从父结点中删除关键字keyList[m - 1]
                    for(var j = m - 1; j < subRoot.keyNumber; j++){
                        subRoot.keyList[j] = subRoot.keyList[j + 1];
                    }
                    subRoot.keyList.pop();

                    // 从父结点中删除结点target
                    for(var j = m; j <= subRoot.keyNumber; j++){
                        subRoot.pointList[j] = subRoot.pointList[j + 1];
                    }
                    subRoot.pointList.pop();
                    subRoot.keyNumber--;

                    target = y;
                    if(subRoot.keyNumber == 0){
                        this.root = subRoot.pointList[1];
                        subRoot = null;
                    }

                    // 从合并后的结点中删除k
                    return this.delete(target, k);
                }
            }

            // 如果子节点在中间
            else if(m > 1 && m <= target.keyNumber){
                // 合并target结点、父结点的keyList[m]、z结点
                if(z != null && z.keyNumber == t - 1){
                    // 合并
                    target.keyList.push(subRoot.keyList[m]);
                    for(var j = 1; j <= z.keyNumber; j++){
                        target.keyList.push(z.keyList[j]);
                    }

                    if(!z.isLeaf){
                        for(var j = 1; j <= z.keyNumber + 1; j++){
                            target.pointList.push(z.pointList[j]);
                        }
                    }
                    target.keyNumber += (1 + z.keyNumber);

                    // 从父结点中删除关键字keyList[m]
                    for(var j = m; j < subRoot.keyNumber; j++){
                        subRoot.keyList[j] = subRoot.keyList[j + 1];
                    }
                    subRoot.keyList.pop();

                    // 从父结点中删除结点z
                    for(var j = m + 1; j <= subRoot.keyNumber; j++){
                        subRoot.pointList[j] = subRoot.pointList[j + 1];
                    }
                    subRoot.pointList.pop();
                    subRoot.keyNumber--;

                    if(subRoot.keyNumber == 0){
                        this.root = subRoot.pointList[1];
                        subRoot = null;
                    }

                    // 从合并后的结点中删除k
                    return this.delete(target, k);
                }

                // 合并y、父节点的keyList[m]、target结点
                else if(y != null && y.keyNumber == t - 1){
                    // 合并
                    y.keyList.push(subRoot.keyList[m]);
                    for(var j = 1; j <= target.keyNumber; j++){
                        y.keyList.push(target.keyList[j]);
                    }

                    if(!target.isLeaf){
                        for(var j = 1; j <= target.keyNumber + 1; j++){
                            y.pointList.push(target.pointList[j]);
                        }
                    }
                    y.keyNumber += (1 + target.keyNumber);

                    // 从父结点中删除关键字keyList[m]
                    for(var j = m; j < subRoot.keyNumber; j++){
                        subRoot.keyList[j] = subRoot.keyList[j + 1];
                    }
                    subRoot.keyList.pop();

                    // 从父结点中删除结点target
                    for(var j = m + 1; j <= subRoot.keyNumber; j++){
                        subRoot.pointList[j] = subRoot.pointList[j + 1];
                    }
                    subRoot.pointList.pop();
                    subRoot.keyNumber--;


                    target = y;
                    if(subRoot.keyNumber == 0){
                        this.root = subRoot.pointList[1];
                        subRoot = null;
                    }

                    // 从合并后的结点中删除k
                    return this.delete(target, k);
                }
            }
        }
        else{
            return this.delete(target, k);
        }
    }
}
```


## 找出最大的元素
伪代码：
```pesudocode
B-TREE-FINDMAX(x):
    if x.isLeaf == true:
        return x.key[x.n];
    else
        return B-TREE-FINDMAX(x.point[x.n + 1]);
```

## 找出最小元素
伪代码：
```pesudocode
B-TREE-FINDMIN(x):
    if x.isLeaf == true:
        return x.key[1];
    else
        return B-TREE-FINDMIN(x.point[1];)
```

## 删除最大元素
伪代码：
```pesudocode
B-TREE-DELTEMAX(x):
    if x.isLeaf == true:
        x.key.pop();
        x.n--;
    else 
        B-TREE-DELTEMAX(x.point[x.n + 1]);
```

## 删除最小元素
伪代码：
```pesudocode
B-TREE-DELETEMIN(x):
    if x.isLeaf == true:
        for i = 1 to x.n - 1:
            x.key[i] = x.key[i + 1];
        x.key[i] = null;
        x.n--;
    else:
        B-TREE-DELETEMIN(x.point[1])
```