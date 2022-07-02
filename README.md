# protocol.js - Protocol v3 JavaScript API

![CoverageStatus](https://github.com/shithuberc/protocol.js/workflows/Coverage/badge.svg)

## Install
```
npm install @shithuberc/protocol.js
```

## Quick start

1. Goto trade.shithub.app to open some positions.

2. Connect to Arbitrum Testnet:

```js
import { JsonRpcProvider } from '@ethersproject/providers'

const provider = new JsonRpcProvider('https://rinkeby.arbitrum.io/rpc')
const chainId = (await provider.getNetwork()).chainId
```

3. Enum Trader's positions (in all perpetual markets). A perpetual is identified by (liquidityPoolAddress, perpetualIndex):

```js
import { CHAIN_ID_TO_POOL_CREATOR_ADDRESS, PoolCreatorFactory } from '@shithuberc/protocol.js'
import { listActivatePerpetualsOfTrader } from '@shithuberc/protocol.js'

const traderAddress = '' // Paste your ETH address here
const poolCreator = PoolCreatorFactory.connect(CHAIN_ID_TO_POOL_CREATOR_ADDRESS[chainId], provider)
const positions = await listActivatePerpetualsOfTrader(poolCreator, traderAddress)
positions.forEach(async ({liquidityPoolAddress, perpetualIndex}) => {
  console.log(liquidityPoolAddress, perpetualIndex)
})
```

4. Get the symbol, underlying name and collateral name of a perpetual. The (ticker) symbol is a number assigned to each perpetual. The underlying name is a string. The collateral is a ERC20 token.
```js
import { getReaderContract, getLiquidityPool } from '@shithuberc/protocol.js'
import { IERC20Factory, erc20Symbol } from '@shithuberc/protocol.js'

const reader = await getReaderContract(provider)
const pool = await getLiquidityPool(reader, liquidityPoolAddress)

const collateralTokenAddress = pool.collateral
const collateral = IERC20Factory.connect(collateralTokenAddress, provider)
const collateralSymbol = await erc20Symbol(collateral)

const perpetual = pool.perpetuals.get(perpetualIndex)
console.log(perpetual.symbol, perpetual.underlyingSymbol, collateralSymbol)
```

5. Get the trader's position size and margin balance. A positive position means buy/long, a negative position means sell/short.
```js
import { BigNumber } from 'bignumber.js'

const account = await reader.callStatic.getAccountStorage(liquidityPoolAddress, perpetualIndex, traderAddress)
console.log(
  'position',
  new BigNumber(account.accountStorage.position.toString()).shiftedBy(-18).toFixed(),
  'margin',
  new BigNumber(account.accountStorage.margin.toString()).shiftedBy(-18).toFixed()
)
```

## Test
```
npm run test
```
