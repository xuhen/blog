---
title: react 使用的 list 最小移动算法
date: 2020-01-17 14:07:17
tags:
---

当我们使用React时，官方都推荐我们给list数据加上key, 这篇文章就是解密为啥加上key 后的列表移动就是最小的了。

下面我们从宏观上看下这个算法的运行规则。

## 带key值的两个数组操作
```
给定两个数组 oldList 和 newList

1. 遍历oldList生成新的数组 simulateList 并判断当前元素是否在newList中，如果不在则向 simulateList 数组加入 null

2. 遍历 simulateList 数组，将元素为null的值标记为 remove 并将其从simulateList删除

3. 遍历 newList 数组，j = i = 0， i 指向newList的索引， j指向simulateList的索引；
    3-1）判断j的索引是否有值，如果为undefind直接将当前的 item 标记为 insert
    3-2）判断当前i,j的索引值是否相等如果相等就 j++, i++
    3-3) 如果i, j的索引值不相等就判断当前item在oldList存在吗，如果不存在就直接将当前item标记为insert
    3-4) 如果当前item在oldList存在，就看当前j+1索引位置的条目和当前i对应的条目相等吗，如果相等，
    就将当前item标记为remove并将当前j对应的条目从simulateList数组删除，然后j++。
    3-5) 如果j+1索引位置的条目和当前i对应的条目不相等，就将i对应的条目标记为insert

4. 最后看j++对应的值是否比simulateList中的个数小，如果是就将j之后的条目标记为remove
```
其中比较难理解的步骤是3-4)，其余部分都比较好理解。

## 算法实现
```
/**
 * Diff two list in O(N).
 * @param {Array} oldList - Original List
 * @param {Array} newList - List After certain insertions, removes, or moves
 * @return {Object} - {moves: <Array>}
 *                  - moves is a list of actions that telling how to remove and insert
 */
function diff (oldList, newList, key) {
  var oldMap = makeKeyIndexAndFree(oldList, key)
  var newMap = makeKeyIndexAndFree(newList, key)
  var newFree = newMap.free
  var oldKeyIndex = oldMap.keyIndex
  var newKeyIndex = newMap.keyIndex
  var moves = []

  // a simulate list to manipulate
  var children = []
  var i = 0
  var item
  var itemKey
  var freeIndex = 0

  // first pass to check item in old list: if it's removed or not
  while (i < oldList.length) {
    item = oldList[i]
    itemKey = getItemKey(item, key)
    if (itemKey) {
      if (!newKeyIndex.hasOwnProperty(itemKey)) {
        children.push(null)
      } else {
        var newItemIndex = newKeyIndex[itemKey]
        console.log('newItemIndex', itemKey, newItemIndex);
        children.push(newList[newItemIndex])
      }
    } else {
      var freeItem = newFree[freeIndex++]
      children.push(freeItem || null)
    }
    i++
  }
  var simulateList = children.slice(0)

  // remove items no longer exist
  i = 0
  while (i < simulateList.length) {
    if (simulateList[i] === null) {
      remove(i)
      removeSimulate(i)
    } else {
      i++
    }
  }

  // i is cursor pointing to a item in new list
  // j is cursor pointing to a item in simulateList
  var j = i = 0
  while (i < newList.length) {
    item = newList[i]
    itemKey = getItemKey(item, key)
    var simulateItem = simulateList[j]
    var simulateItemKey = getItemKey(simulateItem, key)

    if (simulateItem) {
      if (itemKey === simulateItemKey) {
        j++
      } else {
        // new item, just inesrt it
        if (!oldKeyIndex.hasOwnProperty(itemKey)) {
          insert(i, item)
        } else {
          // if remove current simulateItem make item in right place
          // then just remove it
          var nextItemKey = getItemKey(simulateList[j + 1], key)
          if (nextItemKey === itemKey) {
            remove(i)
            removeSimulate(j)
            j++ // after removing, current j is right, just jump to next one
          } else {
            // else insert item
            insert(i, item)
          }
        }
      }
    } else {
      insert(i, item)
    }
    i++
  }

  //if j is not remove to the end, remove all the rest item
  var k = simulateList.length - j
  while (j++ < simulateList.length) {
    k--
    remove(k + i)
  }

  function remove (index) {
    var move = {index: index, type: 0}
    moves.push(move)
  }

  function insert (index, item) {
    var move = {index: index, item: item, type: 1}
    moves.push(move)
  }

  function removeSimulate (index) {
    simulateList.splice(index, 1)
  }

  return {
    moves: moves,
    children: children
  }
}

/**
 * Convert list to key-item keyIndex object.
 * @param {Array} list
 * @param {String|Function} key
 */
function makeKeyIndexAndFree (list, key) {
  var keyIndex = {}
  var free = []
  for (var i = 0, len = list.length; i < len; i++) {
    var item = list[i]
    var itemKey = getItemKey(item, key)
    if (itemKey) {
      keyIndex[itemKey] = i
    } else {
      free.push(item)
    }
  }
  return {
    keyIndex: keyIndex,
    free: free
  }
}

function getItemKey (item, key) {
  if (!item || !key) return void 666
  return typeof key === 'string'
    ? item[key]
    : key(item)
}


// example 1 带 key 的list
var oldList = [{id: "a"}, {id: "b"}, {id: "c"}, {id: "d"}, {id: "e"}]
var newList = [{id: "c"}, {id: "a"}, {id: "b"}, {id: "e"}, {id: "f"}]
var moves = diff(oldList, newList, "id")

moves.moves.forEach(function(move) {
  if (move.type === 0) {
    oldList.splice(move.index, 1) // type 0 is removing
  } else {
    oldList.splice(move.index, 0, move.item) // type 1 is inserting
  }
})

console.log(oldList)


// example 2 没有带key的list
var oldList = [{id: "a"}, {id: "b"}, {id: "c"}, {id: "d"}, {id: "e"}]
var newList = [{id: "c"}, {id: "z"}, {id: "a"}, {id: "b"}, {id: "e"}, {id: "f"}
]


var moves = diff(oldList, newList, function (item) { return item })
moves.moves.forEach(function(move) {
  if (move.type === 0) {
    oldList.splice(move.index, 1) // type 0 is removing
  } else {
    oldList.splice(move.index, 0, move.item) // type 1 is inserting
  }
})

console.log(oldList)


```