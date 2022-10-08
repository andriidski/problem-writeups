---
title: "Basic Tries (Part II) — Word Search Puzzle"
description: "Using tries to implement a 'Word Search' puzzle solver."
---

### Intro 

Back in early school I recall having these assigned on the most random topics as a way to learn keywords or learn English words, or at least I think that was the goal. The premise of the puzzle ([https://en.wikipedia.org/wiki/Word_search](https://en.wikipedia.org/wiki/Word_search)) is simple — you have a grid of what look like random arranged characters (board) in which you are told to find a set of words. The words are "hidden" in a sense that they are surrounded by random characters and the way you "find" the words is look for consecutive streaks of characters on the grid, traditionally in the vertical, horizontal, and diagonal directions. Here is an example grid that I made up with some words on the right that you have to find (and usually circle with a pen)

```
-----------------------
| y s q j u s n a n s |            python
| t d q u p g k u e v |            keywords
| p r f e d z h n g p |            as
| e o e y b e t q g y |            assert
| c w j f l e s p a t |            def
| x y q s o p a s w h |            del
| e e e j z r s o o o |            elif
| z k f i l e k j m n |            else
| l e d u r z z w f z |            except
| f u c t e l m v e c |            for
-----------------------
```

How you would actually solve this by hand is quite intuitive. Given a list of words to search for, pick one word. Then start scanning the grid / board where the word is hidden somewhere. The word can be laid out in any direction, so start trying to match the first character, then the second, then the third, lets say left → right. If there is not a match, repeat but now going top → bottom. If there is no match again, go from the current location in bottom → top direction, and so on. At some point just change the starting location on the board and repeat. What's going on is you effectively are trying to match a series of consecutive characters on the board that comprise a prefix, with the word that you're currently searching for. 

Formally the question can be asked like so

> Given a 2D array of characters comprising a board and given a list of words, search for and return all words from the list that we can successfully find on the board, along with their starting and ending coordinates (this is equivalent to "circling" a word with pen and paper). The words can be laid out on the board in all possible vertical, horizontal, or diagonal directions, but must always follow a straight line (zig-zags are not searchable / counted)

In Python / pseudocode start with something like this

```python
"""
Parameters:
board (List[List[str]]): 2D array / nested list of characters comprising the board
words (List[str]): List of words to search for on the board

Returns:
Dict[str, List[Tuple[int, int]]]: A set of words found on the board along with their 
                                  coordinates. We choose to represent this data as a 
                                  map of word found -> array of two tuples (starting
                                  row,col coordinates and ending row,col coordinates)
"""
def searchForWords(board: List[List[str]], words: List[str]) -> Dict[str, List[Tuple[int, int]]]:
    # TODO: implement
    raise NotImplementedError
```

The return type / structure of course can vary and it's up to the problem creator, but here just return a map of 'found word' → 'coordinates of found word', with 'coordinates of found word' represented by an array of 2 tuples (start & end) with each tuple itself holding the two values for the row & column index, indentifying the position on the 2D board.

For example, the solution to the example board and list of words from earlier

```
-----------------------
| y s q j u s n a n s |            python
| t d q u p g k u e v |            keywords
| p r f e d z h n g p |            as
| e o e y b e t q g y |            assert
| c w j f l e s p a t |            def
| x y q s o p a s w h |            del
| e e e j z r s o o o |            elif
| z k f i l e k j m n |            else
| l e d u r z z w f z |            except
| f u c t e l m v e c |            for
-----------------------
```

 would look like this when printed out

```python
{
    'def': [(2, 4), (2, 2)], 
    'python': [(2, 9), (7, 9)], 
    'else': [(3, 5), (6, 2)], 
    'for': [(4, 3), (6, 5)], 
    'as': [(4, 8), (5, 7)], 
    'assert': [(4, 8), (9, 3)], 
    'except': [(6, 0), (1, 0)], 
    'keywords': [(7, 1), (0, 1)], 
    'elif': [(7, 5), (7, 2)], 
    'del': [(8, 2), (8, 0)]
}
```

Here is an indexed (with axis) version of the grid above for double checking

```python
    0 1 2 3 4 5 6 7 8 9  
  -----------------------
0 | y s q j u s n a n s |
1 | t d q u p g k u e v |
2 | p r f e d z h n g p |
3 | e o e y b e t q g y |
4 | c w j f l e s p a t |
5 | x y q s o p a s w h |
6 | e e e j z r s o o o |
7 | z k f i l e k j m n |
8 | l e d u r z z w f z |
9 | f u c t e l m v e c |
  -----------------------
```

### Walkthrough

This problem is essentially a search problem on an implicit graph — consecutive characters can be thought of as connected with edges to their neighbors in all "legal" directions. Second, as we search the board in different directions, we keep trying to check if what we're looking at is "promising" such that we keep searching, i.e. a prefix of something we're looking at. So really what's needed is to search the grid, while keeping track of seen characters & simulateneously have a way to query if the seen characters are something we are looking for.

Can break it down as so

1. We are given a list of words to search for, i.e. this is our "dictionary". We can build a `Trie` out of these words to have a way to lookup whether a given set of characters is a valid prefix in the Trie. This is useful when we recursively search the board and keep appending characters with every call
2. To search the board / grid, we loop over every cell (`i`, `j` coordinate) and kick off a DFS in all directions. As part of the recursive search we keep track of the current set of characters we are looking at

    During the recursion, some of the state we need to keep track of is which direction we are "currently" going in, i.e. left → right, top → bottom, diagonal bottom-left → top-right, etc. This is needed in order to not search in every possible direction on subsequent calls. We want to kick off the search in all directions initially, but once we do, we record the direction and on the next recursive call will be able to continue in whatever the direction the search is going in

    Here we also can keep track of the starting coordinates, since this is the point where the search begins, which means this is where we can potentially beging to match one of the words. 

3. The way to explore the board / do the search is by incrementing / decrementing the index of either row or column by one. Every legal direction is just two numbers for the direction, for instance right → left update could be [0, -1] and bottom → top [-1, 0].

4. The recursion should terminate out-of-bounds, i.e. during a "step" into a certain direction.

5. Finally, we need a way to detect when we find a word on the board — for this we can use the Trie built using the words that we are looking for. Suppose we have a function to return an actual Trie node given a string prefix, then on every recursive call we call this function using the current sequence of characters accumulated and get the node that corresponds to this prefix. 

    Now check this node — if it is not defined / `None`, then we know we should stop the recursion, if it is a valid node, there are three cases

    1. Node is a "leaf" / "word" node meaning it is a node that was marked as a word when inserting, in which case we found one of the words we are searching for
    2. Node is one of the internal nodes, in which case we want to keep searching
        1. If the direction in which we are searching is not specified, that means that this is the initial search call from the current cell, so we want to recursively search in every valid direction. We can build a map of directions → coordinate updates that provide us with a state update for the next search call, then iterate this map and kick off a search in every direction
        2. If the direction is specified, then we just want to continue down this direction. In this case we only need to lookup the coordinate updates for the direction in which we're searching and call our search function in that direction

```python
"""
Our Trie data structure. 
"""
class TrieNode:
    def __init__(self, isWord=0):
        self.children = {}
        self.isWord = isWord
    
    def insert(self, word):
        curr = self
        
        for ch in word:
            if ch in curr.children:
                curr = curr.children[ch]
            else:
                node = TrieNode()
                curr.children[ch] = node
                curr = node
        
        curr.isWord = 1

    def getNodeForPrefix(self, prefix):
        curr = self
        
        for ch in prefix:
            if ch in curr.children:
                curr = curr.children[ch]
            else:
                return None
            
        return curr
    
class Solution:
    """
    Parameters:
    board (List[List[str]]): 2D array / nested list of characters comprising the board
    words (List[str]): List of words to search for on the board

    Returns:
    Dict[str, List[Tuple[int, int]]]: A set of words found on the board along with their 
                                      coordinates. We choose to represent this data as a 
                                      map of word found -> array of two tuples (starting
                                      row,col coordinates and ending row,col coordinates)
    """
    def findWords(self, board: List[List[str]], words: List[str]) -> Dict[str, List[Tuple[int, int]]]:

        # All the words that we collect.
        self.ans = {}

        # 1. Build the Trie from the words that comprise our "dictionary".
        trie = TrieNode()
        for word in words:
            trie.insert(word)
            
        # 2. Search the board by looping over all cells and kick off a depth-first search in all directions.
        #    During the recursion we need to keep track of state such as the current direction, the current
        #    prefix, the current location on the board, and whatever the initial starting position is.
        #    The direction is needed since legal word arrangements cannot be stored in zig-zags, and the 
        #    initial position is needed since at some later recursive call we might find one of the words
        #    we're looking for and need to return this information for the coordinates of the word found on
        #    the board.

        n = len(board)
        m = len(board[0])
        
        for i in range(0, n):
            for j in range(0, m):
                # The initial call to kick off the search.
                self.explore(i, 
                             j, 
                             None, # Initial direction is not specified initially.
                             None, # We will set the starting row coordinate at the initial call. 
                             None, # ^ Same for col coordinate.
                             n, 
                             m, 
                             board, 
                             trie, 
                             ''    # Initially prefix is empty, we haven't 'seen' any cells yet.
                )
        
        return self.ans
        
    
    def explore(self, i, j, direction, starting_i, starting_j, n, m, board, trie, prefix):
        # 3. One of the conditions to stop the search / exploration is when the search goes
        #    out-of-bounds of the board, in which case we just return.
        if i < 0 or i >= n or j < 0 or j >= m:
            return
        
        # Another case is when we come back to the cell which we've started from. We implement
        # this by marking a cell with some special character when we start the traversal and 
        # then check the current cell which we are searching here.
        if board[i][j] == "#":
            return
        
        currentLetter = board[i][j]
        
        # 4. If we are at this point in the explore() call, then we are inside valid bounds of 
        #    the board and want to check whatever the current prefix is to see if it leading us
        #    in the right direction. So we query our Trie built from words that we are searching
        #    for in order to get a node (if any) that would correspond to the node at the end of
        #    the current prefix. The current prefix is whatever the current running prefix is that
        #    we keep track of as we search the grid with whatever the current letter at the current
        #    cell position is.
        #
        #    For example, if the Trie looks like this
        #    
        #                     root
        #                      |
        #                     / \ 
        #                    a  b
        #                    |
        #                    c
        #
        #    Then the query for prefix 'ac' would return the node linked to by 'c' (leaf node in this
        #    case), but if the the current prefix would be 'acb' then it would return the value None.
        #    Note that because of the properties of the Trie, this lookup is O(length of prefix + 1).

        currentNode = trie.getNodeForPrefix(prefix + currentLetter)
        
        # 5. Based on what the current node is that we query from the Trie, we need to proceed
        #    differently. If the node is None, then we really don't do anything and want to stop the
        #    search, since that would mean that whatever direction we have now searched in, does not
        #    correspond to any prefix or word that is stored in the Trie.
        #    
        #    If the node is valid and not None, then we have three scenarios. First, if the node
        #    is marked as a 'word' node (recall we do this when inserting intro the Trie), then we
        #    know that we've found one of the words that we're looking for. In that case, we want to
        #    record the data that we are asked for, and then we also need to mark this node as no
        #    longer a 'word node'. This is needed to prevent duplicates, since we are likely to keep
        #    searching and might find this match from querying the Trie again. If the node is not 
        #    marked, then it is one of the internal nodes, so we know we need to continue searching. 
        #
        #    The way we keep searching depends on the value of the 'direction'. Recall that we need 
        #    to follow a single direction once we commit to one in order to avoid zig-zags, and we
        #    initially started with direction set to None. So, if the direction is not yet set, then
        #    we want to continue the search in all directions. We can do this by building a map of
        #    relative direction name (this is just for helping orient) to a tuple of coordinate update
        #    values that essentially dictate what the state of the search call will be on the next
        #    call, akin to following an edge in a graph. We then iterate this map, and continue the
        #    the search by calling explore(), but now we are committed to the direction, so we also
        #    make sure to specify the direction and the starting row, col coordinates.
        #    
        #    Finally, if the direction is set, then instead of making many 'strides' in different
        #    directions, we just have to continue in the direction that we are already going in. So
        #    we just lookup the direction from the same map that we've built for the earlier case,
        #    and make a single call to keep exploring in this direction. Note that this would mean
        #    that the starting_i and starting_j coordinates were set sometime earlier in the traversal,
        #    so we simply pass those down to the recursive call.

        if currentNode and currentNode.isWord == 1:
            self.ans[prefix + currentLetter] = [(starting_i, starting_j), (i, j)]
            currentNode.isWord = 0
        
        if currentNode:
            # Mark the current start cell, so that if we see it later in traversal we would know that
            # we've come back to the original letter and should stop, since we can't use the same 
            # letter more than once.
            board[i][j] = '#'

            # The map of all directions that we can initiate a search in and in which directions the
            # words hidden on the board can be aligned in.
            all_dirs = {
                'left': [0, -1],
                'right': [0, 1],
                'up': [-1, 0],
                'down': [1, 0],
                'diag_left_up': [-1, -1],
                'diag_left_down': [-1, 1],
                'diag_right_up': [1, -1],
                'diag_right_down': [1, 1],
            }

            # If a specific direction is not specified, start the search in all directions, otherwise
            # follow a single direction by looking up the coordinates which we need to use to offset
            # the current position on the grid by.
            if not direction:
                for direction_key in all_dirs:
                    coords = all_dirs[direction_key]
                    self.explore(i + coords[0], 
                                 j + coords[1], 
                                 direction_key,
                                 i,
                                 j, 
                                 n, 
                                 m, 
                                 board, 
                                 trie, 
                                 prefix + currentLetter # Extend the prefix with current letter.
                    )
            else:
                coords = all_dirs[direction]
                self.explore(i + coords[0], 
                             j + coords[1], 
                             direction, 
                             starting_i, 
                             starting_j, 
                             n, 
                             m, 
                             board, 
                             trie, 
                             prefix + currentLetter # Extend the prefix with current letter.
                )
        
        # If we are here then we've searched to a cell and built a prefix that is not present in the
        # Trie, so we set the cell on the board back to the original letter.
        board[i][j] = currentLetter
        
        return
```

One thing that's interesting here is performance. And related to that — do you need a trie and what about just a set to lookup the prefixes.

On my machine solving the example grid and list of words from earlier is instant. A grid of size `100x100` is also a fraction of a second. A `1000x1000` grid with a mix of small and medium length words to search for like 

```python
['python', 'code', 'algorithm', 'dfs', 'bfs', 'loop', 'recursion']
```
runs in about __2.5-3.0 seconds__ on my machine.

For the second part, the modifications that one would have to make are pretty minor. Just replace the trie building with a simple construction of a set while keeping track of a longest word like so

```python
hashset = set()
longest_word = 0
for word in words:
    hashset.add(word)
    longest_word = max(longest_word, len(word))
```

then instead of passing the trie into the search, pass the set. We don't have a way to lookup nodes now, so we modify the code to check if the current prefix on the recursive call is contained within the set. If yes, we found one of the words, if no, then we continue searching until we're either out of bounds or the prefix is too long (longer than the max word stored in the set). So add a return case

```python
if len(prefix) > longest_word:
    return
```

This helps with early stopping, and then the check for if the current prefix is stored in the hashset will look like this

```python
if prefix + currentLetter in hashset:
    self.ans[prefix + currentLetter] = [(starting_i, starting_j), (i, j)]
    return
else:
    # Do everything else to continue searching.
```

I used the same list of words with pretty medium-length words, then added a longer word and changed the size of the board. The table with a bunch of the measurements on my machine is below, but just to give an example, on a `10x10` character board the two solutions are of course equivalent. On a `250x250` character board, which is really not even that large, the trie version finishes in about __0.2 seconds__ at most, while the set in around __5.5-6.0 seconds__. Thats about 30x slower. If you add a word thats longer, say `"depthfirstsearchtraversalalgorithm"` (34 characters), the running time jumps up to about __20 seconds__, while using the trie stays the same (why this is the case can be a follow-up about lookup time in a trie and what is the worst case time complexity there). For boards around `1000x1000` characters, without a trie you will start erroring out with "maximum recursion depth exceeded in comparison". But even for `250x250` this is already around 100x slower.

| Prefix Lookup | Board Size | Running Time (seconds) |
|---------------| ---------- |:----------------------:|
|`Trie`         | 10x10      | 0.001                  |
|`Trie`         | 100x100    | 0.029                  |
|`Trie`         | 250x250    | 0.163                  |
|`Trie`         | 500x500    | 0.628                  |
|`Trie`         | 1000x1000  | 2.586                  |
|-              |-           |-                       |
|`Set`          | 10x10      | 0.005                  |
|`Set`          | 100x100    | 0.813                  |
|`Set`          | 250x250    | 5.422                  |
|`Set`          | 500x500    | 21.671                 |
|`Set`          | 1000x1000  | N/A                    |
