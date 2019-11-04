# block_chain
blockchain example

学习区块链--构建一个

最快的方式来学习区块链是构建一个区块链。

看到这篇文章， 你一定像我一样对数字加密货币的高速发展感到特别兴奋。你也想知道加密技术的基本(区块链)是怎样工作的。

至少对于我来说， 理解区块链是不容易的， 我通过观看大量的视频、 专题报告来学习， 并且处理几个示例里面的问题。

我喜欢在实战中学习， 它强迫我在代码层面处理事情， 那样我会坚持。如果你也这样做， 在本文最后， 机会得到一个可以工作的区块链， 同时会牢牢掌握区块链的运行原理。

开始之前

区块链是不可变的、 有序的数据记录链， 数据记录成为区块。区块可以包含交易、文件等任何你想要的数据，他们之间通过hash来链接在一起。

这篇文章的目标对象是谁? 可以阅读并且写一些简单的python， 并且理解http请求， 因为讨论的区块链是基于http的。

需要那些工具 python3.6+、 pip必须安装， 还需要安装Flash pip install Flask==0.12.2 requests==2.18.4，还需要http客户端， 比如Postman、 Curl。

第一步： 创建一个区块链

打开你最喜欢的IDE编辑器， 比如vscode， 穿件一个新文件blockchain.py， 现在只创建一个文件， 如果有问题， 你可以一直参照源码：
https://github.com/dvf/blockchain

展示区块链

先创建一个blockchain类， 这个构造函数初始化一个控列表来存储我们的区块链， 和其他的交易， 这是类的构造：
```
class Blockchain(object):
    def init(self):
        self.chain = []
        self.current_transactions = []
    def new_block(self):
    # Creates a new Block and adds it to the chain
    pass

def new_transaction(self):
    # Adds a new transaction to the list of transactions
    pass

@staticmethod
def hash(block):
    # Hashes a Block
    pass

@property
def last_block(self):
    # Returns the last Block in the chain
    pass
```

我们的blockchain类负责管理链， 它会存储一些交易并且为添加区块提供一些辅助方法， 让我们丰富我们的一些方法。

一个区块应该是什么样子?

每一个区块链有一个索引， 一个时间戳， 一个交易列表， 一个证明（后面介绍）， 和前一个区块的hash值。

比如这个结构：
```
block = {
    'index': 1,
    'timestamp': 1506057125.900785,
    'transactions': [
        {
            'sender': "8527147fe1f5426f9dd545de4b27ee00",
            'recipient': "a77f5cdfa2934df3954a5c7c7da5df1f",
            'amount': 5,
        }
    ],
    'proof': 324984774000,
    'previous_hash': "2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824"
}
```
这时候， 连续这个概念应该可以理解了， 每一个新的区块包含先前区块的hash值。这点是非常重要的， 保证了区块链的不易改变性， 如果一个攻击者破坏先前的区块， 那么包含这个区块后面所有的区块都会被破坏。

这样是否有意义？如果不这这样做， 花费一些时间是可以破解区块链。

给交易里面添加区块

我们需要一个方式来往区块里面添加交易， new_transaction()方法负责这个， 并且它很简单
```
class Blockchain(object):
    ...

    def new_transaction(self, sender, recipient, amount):
        """
        Creates a new transaction to go into the next mined Block
        :param sender: <str> Address of the Sender
        :param recipient: <str> Address of the Recipient
        :param amount: <int> Amount
        :return: <int> The index of the Block that will hold this transaction
        """

        self.current_transactions.append({
            'sender': sender,
            'recipient': recipient,
            'amount': amount,
        })

        return self.last_block['index'] + 1
```
在new_transaction() 在列表里面添加交易之后， 它返回交易即将所在区块的index， 也就是写一个区块， 这将会在后来有用， 用来提交用户的交易。

创建新区块

当我们区块链实例化之后， 我们需要生成一个创世区块， 该区块没有前区块。我们仍然需要添加为创世区块添加一个证明， 证明的过程就是挖矿， 稍后解释。

除了给构造函数添加创世区块外， 还需要丰富我们new_block, new_transaction,hash函数。
```
import hashlib
import json
from time import time

class Blockchain(object):
    def __init__(self):
        self.current_transactions = []
        self.chain = []

        # Create the genesis block
        self.new_block(previous_hash=1, proof=100)

    def new_block(self, proof, previous_hash=None):
        """
        Create a new Block in the Blockchain
        :param proof: <int> The proof given by the Proof of Work algorithm
        :param previous_hash: (Optional) <str> Hash of previous Block
        :return: <dict> New Block
        """

        block = {
            'index': len(self.chain) + 1,
            'timestamp': time(),
            'transactions': self.current_transactions,
            'proof': proof,
            'previous_hash': previous_hash or self.hash(self.chain[-1]),
        }

        # Reset the current list of transactions
        self.current_transactions = []

        self.chain.append(block)
        return block

    def new_transaction(self, sender, recipient, amount):
        """
        Creates a new transaction to go into the next mined Block
        :param sender: <str> Address of the Sender
        :param recipient: <str> Address of the Recipient
        :param amount: <int> Amount
        :return: <int> The index of the Block that will hold this transaction
        """
        self.current_transactions.append({
            'sender': sender,
            'recipient': recipient,
            'amount': amount,
        })

        return self.last_block['index'] + 1

    @property
    def last_block(self):
        return self.chain[-1]

    @staticmethod
    def hash(block):
        """
        Creates a SHA-256 hash of a Block
        :param block: <dict> Block
        :return: <str>
        """

        # We must make sure that the Dictionary is Ordered, or we'll have inconsistent hashes
        block_string = json.dumps(block, sort_keys=True).encode()
        return hashlib.sha256(block_string).hexdigest()
```
上面比较好理解， 我添加了一些注释和文档来让它更清楚， 我们现在完成了代表我们的区块链。在这个点上， 你一定想知道新的区块是怎样创建， 跟挖矿的。

理解工作证明

工作证明(Proof of Work -- POW)算法是怎样创建一个新的区块或者挖矿的呢？POW的是找到一个数字来解决这个问题的。这个数字是很难到的但是很容易被证明是正确的。这是POW的核心。

我们看一个简单的例子来了解它。

如果我们假设整数x成y必须是以0结尾的hash， 也就是 hash（x*y) = ac23dc...0， 举一个简单的例子， 我们修改x=5， 在python工具里面：
```
from hashlib import sha256
x = 5
y = 0  
# We don't know what y should be yet...
while sha256(f'{x*y}'.encode()).hexdigest()[-1] != "0":
    y += 1
print('The solution is y = {y}')
```
如果找到满足最后为0的y值， y的值为21：
`hash(5 * 21) = 1253e9373e...5e3600155e860`

在比特币里面， 工作量证明算法被成为Hashcash， 它跟我们上面的演示差不多， 旷工竞赛算法就是为了产生一个新的区块， 通常来说， 困难的程度是由一个需要查找的字符串决定的， 如果旷工找到这个字符串将会在这个交易里获得一些货币奖励。

其他的旷工是很容易验证他们的解决方案是否正确的。

实现简单的工作证明

我们为区块链实现一个简单的可用算法，规则是类似上面的:
当做hash运算的时候， 寻找到一个p来让该区块的hash值有4个前导0， 就是以0000开头。
```
import hashlib
import json

from time import time
from uuid import uuid4


class Blockchain(object):
    ...

    def proof_of_work(self, last_proof):
        """
        Simple Proof of Work Algorithm:
         - Find a number p' such that hash(pp') contains leading 4 zeroes, where p is the previous p'
         - p is the previous proof, and p' is the new proof
        :param last_proof: <int>
        :return: <int>
        """

        proof = 0
        while self.valid_proof(last_proof, proof) is False:
            proof += 1

        return proof

    @staticmethod
    def valid_proof(last_proof, proof):
        """
        Validates the Proof: Does hash(last_proof, proof) contain 4 leading zeroes?
        :param last_proof: <int> Previous Proof
        :param proof: <int> Current Proof
        :return: <bool> True if correct, False if not.
        """

        guess = f'{last_proof}{proof}'.encode()
        guess_hash = hashlib.sha256(guess).hexdigest()
        return guess_hash[:4] == "0000"
```

可以通过调整前导零的数量来调整算法的困难程度， 但是4是足够的。可以发现， 添加一个前导零会让解决的时间产生巨大的变化。

我们的类几乎已经完成了， 我们现在可以跟他进行交互。

第二步：让区块链可以使用API来调用

我们使用Python的Flask框架来进项开发， 它是一个小型的web框架， 并且可以非常简映射到Python的函数。然后就可以使用http请求来访问了。

我们将会创建三个方法：
/transactions/new 创建一个新的交易区块
/mine 用来告诉服务用来挖一个区块
/chain 返回整个区块

创建Flask

我们的服务会在我们的区块链中形成一个单一的节点， 我们先写一些引用模板：
```
import hashlib
import json
from textwrap import dedent
from time import time
from uuid import uuid4

from flask import Flask


class Blockchain(object):
    
# Instantiate our Node
app = Flask(__name__)

# Generate a globally unique address for this node
node_identifier = str(uuid4()).replace('-', '')

# Instantiate the Blockchain
blockchain = Blockchain()

@app.route('/mine', methods=['GET'])
def mine():
    return "We'll mine a new Block"

@app.route('/transactions/new', methods=['POST'])
def new_transaction():
    return "We'll add a new transaction"

@app.route('/chain', methods=['GET'])
def full_chain():
    response = {
        'chain': blockchain.chain,
        'length': len(blockchain.chain),
    }
    return jsonify(response), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```
如果Flash那有问题， 可以看下看下官网。

交易端

每一个请求的格式应该是这个样子:
```
{
 "sender": "my address",
 "recipient": "someone else's address",
 "amount": 5
}
```
因为我们已经完成了添加交易的方法， 接下来会非常简单， 先来添加一个交易：
```
import hashlib
import json
from textwrap import dedent
from time import time
from uuid import uuid4
from flask import Flask, jsonify, request

@app.route('/transactions/new', methods=['POST'])
def new_transaction():
    values = request.get_json()

    # Check that the required fields are in the POST'ed data
    required = ['sender', 'recipient', 'amount']
    if not all(k in values for k in required):
        return 'Missing values', 400

    # Create a new Transaction
    index = blockchain.new_transaction(values['sender'], values['recipient'], values['amount'])

    response = {'message': f'Transaction will be added to Block {index}'}
    return jsonify(response), 201
```
挖矿端

挖矿端会有神奇的事情发生， 并且很简单， 需要做下面三个事情：
工作证明计算
给添加交易的旷工一个币
将区块加入到链里面
```
import hashlib
import json
from time import time
from uuid import uuid4
from flask import Flask, jsonify, request

@app.route('/mine', methods=['GET'])
def mine():
    # We run the proof of work algorithm to get the next proof...
    last_block = blockchain.last_block
    last_proof = last_block['proof']
    proof = blockchain.proof_of_work(last_proof)

    # We must receive a reward for finding the proof.
    # The sender is "0" to signify that this node has mined a new coin.
    blockchain.new_transaction(
        sender="0",
        recipient=node_identifier,
        amount=1,
    )

    # Forge the new Block by adding it to the chain
    previous_hash = blockchain.hash(last_block)
    block = blockchain.new_block(proof, previous_hash)

    response = {
        'message': "New Block Forged",
        'index': block['index'],
        'transactions': block['transactions'],
        'proof': block['proof'],
        'previous_hash': block['previous_hash'],
    }
    return jsonify(response), 200
```
被开采的块的接受者是我们的节点地址， 并且我们做的大多数的试仅仅是跟我们的Blockchain类在做交互。现在 我们做完了， 并且可以开始交互了。

第三步：跟区块链进行交互

我们可以使用Postman跟Curl来进行交互。

首先开启服务：
```
$python blockchain.py
* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```
实现功能介绍：

方法| URL| 参数| 作用

|--|--|--|--|
|get|http://localhost:5000/mine| 无| 开始挖矿
|post| http://localhost:5000/transactions/new|   "sender":"发送币的地址",
"recipient": "接受币的地址",
"amount": 币的数量| 发起一个交易|
|get|http://localhost:5000/chain|无| 获取整个链的交易|

当使用curl http://localhost:5000/mine的时候， 服务器会开始挖矿， 如果没有人发起交易， 这里面会只有一条交易， 该交易是奖励挖矿人的， recipient即为挖矿人的地址。

下面这个接口
```
$ curl -X POST -H "Content-Type: application/json" -d '{
 "sender": "d4ee26eee15148ee92c6cd394edd974e",
 "recipient": "someone-other-address",
 "amount": 5
}' "http://localhost:5000/transactions/new"
```
会生成一天交易， 该交易存放在即将打包的区块中， 当使用挖矿接口挖矿的时候， 该交易会被打包到区块链里面。

当使用curl http://localhost:5000/chain时， 该命令返回返回区块链里面的所有数据。

第四步：共识算法

我们现在可以接受一个交易并且将交易打包成一个新的区块， 但是我们的系统应该是分散管理的。不应该以一个服务为主。如果分散管理， 怎样保证他们使用的同一个连呢？这个问题叫做共识，我们将会写一个共识算法来让我们的多个节点工作。

注册一个新的节点

在开始实现共识算法之前，我们需要让别的节点知道有一个新的节点到达我们的网络上了。我们网络中每一个节点应该有这个网络中全部注册是节点， 因此，我们需要一些路由（endpoints）：

/nodes/register 用来获取一些新的节点，格式是URL
/nodes/resolve 用来实现共识算法， 解决任何冲突， 保证链的正确性。
我们需要修改我们的构造函数， 并且提供注册节点的方法。
```
...
from urllib.parse import urlparse
...

class Blockchain(object):
    def __init__(self):
        ...
        self.nodes = set()
        ...

    def register_node(self, address):
        """
        Add a new node to the list of nodes
        :param address: <str> Address of node. Eg. 'http://192.168.0.5:5000'
        :return: None
        """

        parsed_url = urlparse(address)
        self.nodes.add(parsed_url.netloc)
```
注意， 我们使用set()来获取节点的列表，这是确保我们列表在多次插入之后只有一次的最简单的方式。

编写共识算法

正如我们说的， 冲突是指两个节点直接出现不一样的链。为了解决这个问题， 我们指定一个规则：最长的链既为正确的有权威的链， 也就是说链的长度是一个判定标准， 使用这个算法， 我们将共识写入到我们的节点中。
```

import requests


class Blockchain(object)
    ...

    def valid_chain(self, chain):
        """
        Determine if a given blockchain is valid
        :param chain: <list> A blockchain
        :return: <bool> True if valid, False if not
        """

        last_block = chain[0]
        current_index = 1

        while current_index < len(chain):
            block = chain[current_index]
            print(f'{last_block}')
            print(f'{block}')
            print("\n-----------\n")
            # Check that the hash of the block is correct
            if block['previous_hash'] != self.hash(last_block):
                return False

            # Check that the Proof of Work is correct
            if not self.valid_proof(last_block['proof'], block['proof']):
                return False

            last_block = block
            current_index += 1

        return True

    def resolve_conflicts(self):
        """
        This is our Consensus Algorithm, it resolves conflicts
        by replacing our chain with the longest one in the network.
        :return: <bool> True if our chain was replaced, False if not
        """

        neighbours = self.nodes
        new_chain = None

        # We're only looking for chains longer than ours
        max_length = len(self.chain)

        # Grab and verify the chains from all the nodes in our network
        for node in neighbours:
            response = requests.get(f'http://{node}/chain')

            if response.status_code == 200:
                length = response.json()['length']
                chain = response.json()['chain']

                # Check if the length is longer and the chain is valid
                if length > max_length and self.valid_chain(chain):
                    max_length = length
                    new_chain = chain

        # Replace our chain if we discovered a new, valid chain longer than ours
        if new_chain:
            self.chain = new_chain
            return True

        return False
```
valid_chain()是用来检测一个链是不是有效的， 通过循环认证hash跟证明（proof）

resolve_conflicts()该方法通过循环我们附近的所有节点， 下载他们的链并且认证他们， 如果一个最长的有效的链被找见， 用它来代替我们的。

我们注册两个路由(endpoints)在我们的API里面， 一个为了添加结点， 一个为了解决冲突。
```
@app.route('/nodes/register', methods=['POST'])

def register_nodes():
    values = request.get_json()

    nodes = values.get('nodes')
    if nodes is None:
        return "Error: Please supply a valid list of nodes", 400

    for node in nodes:
        blockchain.register_node(node)

    response = {
        'message': 'New nodes have been added',
        'total_nodes': list(blockchain.nodes),
    }
    return jsonify(response), 201



@app.route('/nodes/resolve', methods=['GET'])

def consensus():
    replaced = blockchain.resolve_conflicts()

    if replaced:
        response = {
            'message': 'Our chain was replaced',
            'new_chain': blockchain.chain
        }
    else:
        response = {
            'message': 'Our chain is authoritative',
            'chain': blockchain.chain
        }

    return jsonify(response), 200
```
这时候你可以访问不同的结点， 并且在你的网络上部署结点。或者使用相同的电脑但是使用不同的端口来实现， 我是使用两个不同的端口来实现的http://localhost:5000和http://localhost:5001

然后在一个结点里面进行挖矿， 就是调用mine接口， 多挖几个， 确保它的链是最长的， 之后， 调用GET /node/resolve 在另一个节点上， 另一个节点的链将会通过共识算法被替换。

译文： 
[https://hackernoon.com/learn-blockchains-by-building-one-117428612f46](https://hackernoon.com/learn-blockchains-by-building-one-117428612f46)
