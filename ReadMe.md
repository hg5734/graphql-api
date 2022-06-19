# Graph Query API Service for DEX operation like Add liquidity, Remove liquidity, swap etc

---
# component 
1. **Blocklistener MicroService** -> this service listen blockchain block and send to **blockparser** via queues.

```typescript
  web3ObjSocket.eth.subscribe('newBlockHeaders',(error) =>{
      .....
  })

  ```


2. **Blockparser MicroService** -> this service listen queues of **block-listener** and parse the block events of required smart contract and prepare data and send to **GraphDataManager** service to store data in graph.

```typescript
    await web3ObjHttp.eth.getPastLogs({
      toBlock: web3ObjHttp.utils.toHex(toBlock),
      fromBlock: web3ObjHttp.utils.toHex(fromBlock),
      address: contractAddress
    });

    Events 
    1. addLiquidity (address tokenA, address tokenB, uint256 amountADesired, uint256 amountBDesired, uint256 amountAMin, uint256 amountBMin, address to, uint256 deadline)
    // https://etherscan.io/tx/0xeb189691a95c1b1d1fdba462d68315c81afd31c20cd481bd298b03e978875d7f/
    2. removeLiquidity(address tokenA, address tokenB, uint256 liquidity, uint256 amountAMin, uint256 amountBMin, address to, uint256 deadline, bool approveMax)
    //https://etherscan.io/tx/0x17b95ee8c9c24102a3ade30ad24e5574f61a950686d66a5ce1bc9cb3bdfaddca/
    3. swap(uint256 amount0Out, uint256 amount1Out, address to, bytes data)
    // https://etherscan.io/tx/0xa6e4fa915fa732eafde4f40246819e72121fbb29e7fd8af09cb11debefe7ef09
  ```

3. **GraphDataManager MicroService** -> this service listen queues of **Blockparser** and store data in database in required format of based on graph schema. 


  ```typescript

  var express = require('express');
  var { graphqlHTTP } = require('express-graphql');
  var { buildSchema } = require('graphql');
  var schema = buildSchema(`
    type LiquidityPool {
    }
    `);

  // The root provides a resolver function for each API endpoint
  var root = {
  };
  
  var app = express();
  app.use('/graphql', graphqlHTTP({
    schema: schema,
    rootValue: root,
    graphiql: true,
  }));
  app.listen(4000);
  console.log('Running a GraphQL API server at http://localhost:4000/graphql')

 ```

4. **Schema Design for Dex operation** -> Schema Design for storing data for liquidity provider and user positions in liquidity pool.

  ```typescript
     type LiquidityPool {
         token1: String;
         token2: String;
         contractAddress: String;
         totalSupply: Int;
         .....
     }

     type User {
          id: ID!
          address: String!
          balance: BigInt!
          transactionCount: Int!
          ......
      }
   ```

5. **Scaling** -> We can scale up/down our services using EKS(Elastic Kubernetes Services). (https://aws.amazon.com/eks/)
