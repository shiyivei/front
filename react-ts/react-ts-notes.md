# 5 状态管理

react-ts中有两种传递数据，一种是数据传递，另一种是传递状态，它们都是通过接口定义

**数据传递**（传递的是参数的名称和附加类型）

这里的数据传递实际上只是定义了类型名称以及类型，数据具体的是什么还需要在渲染这个组件的组件里进行赋值，形式上类似于标签的参数

```
interface IPropos {
  address?: string; //通过问号让类型可选传递，通过接口声明数据类型
  balance: string;
}
```

**状态管理**

传递状态（传递的是真正的值）

```
// 状态管理
interface IState {
  count: number;
}
```

**通过泛型的形式指定**

```
// 2. 以范型的形式绑定类型（这里的propos实际上就值得是接口本身）
export default class Reward extends React.Component<
  IPropos,
  IState
> {
  // 通过constructor 初始化参数
  public constructor(props: any) {
    super(props);
  }

  // 这里是初始化状态,渲染时不用在另一个组件中指定，问题来了初始化以后怎么更新呢
  public readonly state: Readonly<IState> = {
    count: 1000,
  };

  public render() {
    // 3. 调用数据和状态
    const { address } = this.props;
    const { count } = this.state;
    // 4. 参数渲染的时候会像组件的样式一样使用：如<div style = "color: black">
    // 5. 但是这种方式只是传个值过去，具体的值还是要在App,tsx中再指定
    return (
      <>
        <div>{address}</div>
        <div>{count}</div>
      </>
    );
  }
}
```

使用Handler改变状态值

```
import React from "react";
import Web3 from "web3";
import ABI from "./utilis/abi";
import { nanoid } from "nanoid";
import CONTRACT_ADDRESS from "./utilis/contract_address";
import PRIVATE_KEY from "./utilis/env";
import TOKEN_ADDRESS from "./utilis/token_address";
import { getValue } from "@testing-library/user-event/dist/utils";
import "./Reward.css";

const Big = require("big.js");

// 1. 通过接口声明参数(多个)
interface IPropos {
  address?: string; //通过问号让类型可选传递
  balance?: string;
}

// 状态管理
interface IState {
  count: number;
}

// 2. 以范型的形式绑定类型（这里的propos实际上就值得是接口本身）
export default class Reward extends React.Component<
  IPropos,
  IState
> {
  // 通过constructor 初始化参数
  public constructor(props: any) {
    super(props);
    //修改值的时候需要在构造函数中初始化,而不是readonly,this 就指的的本组件,所有的都建议在constructor中声明
    this.state = {
      count: 0,
    };

    //把下面定义的clickHandler绑定给本组件的clickHandler
    this.clickHandler =
      this.clickHandler.bind(this);
  }

  // 这里是初始化状态,渲染时不用在另一个组件中指定，问题来了初始化以后怎么指定呢
  // public readonly state: Readonly<IState> = {
  //   count: 1000,
  // };

  public clickHandler() {
    this.setState({
      count: 2000,
    });
  }

  public render() {
    // 3. 调用数据和状态
    const { address } = this.props;
    const { count } = this.state;
    // 4. 参数渲染的时候会像组件的样式一样使用：如<div style = "color: black">
    // 5. 但是这种方式只是传个值过去，具体的值还是要在App,tsx中再指定
    return (
      <>
        <div>{address}</div>
        <div>{count}</div>
        <div className="Change-balance">
          <button onClick={this.clickHandler}>
            查看可提现余额
          </button>
        </div>
      </>
    );
  }
}

console.log(
  "hello, welcome to the reward contract !"
);

// Initialize web3.js using the IoTeX Babel endpoint
const web3 = new Web3(
  new Web3.providers.HttpProvider(
    "https://babel-api.testnet.iotex.io"
  )
);

const contract_address: any = CONTRACT_ADDRESS;

console.log("1.合约地址是:", contract_address);

//合约ABI
const abi: any = ABI;

//合约实例
const reward = new web3.eth.Contract(
  abi,
  contract_address
);

//2.获取合约信息
console.log("2.创建的合约实例:", reward);

//1.查看合约创建者
const get_owner = async () => {
  var owner = await reward.methods
    .contract_owner()
    .call();
  console.log("3.合约创建者:", `${owner}`);

  return owner;
};

get_owner();

//2.查询创建者账户余额
const test_address =
  "0xFb7032b3fcfFc0A41E96B99AFd663A477819667C";

const get_iotex_test_balance = async () => {
  let result = web3.eth
    .getBalance(test_address)
    .then(function (balance: any) {
      let iotxBalance = Big(balance).div(
        10 ** 18
      );
      console.log(
        "4.%s 账户余额: %s IOTX",
        test_address,
        iotxBalance.toFixed(2)
      );
      return iotxBalance.toFixed(2);
    });

  return result;
};

get_iotex_test_balance();

//3.查询当前合约余额

const token_address = TOKEN_ADDRESS;
const get_contractBalance = async () => {
  let result = await reward.methods
    .get_contract_balance(token_address)
    .call();

  let balance: number = result;
  console.log(
    "5.当前合约余额:",
    Big(balance)
      .div(10 ** 18)
      .toFixed(0)
  );
};

get_contractBalance();

//4.查询合约某个报告是否已经存储（出具）

const get_report_hash_store_status = async (
  report_hash: string
) => {
  let result = await reward.methods
    .report_is_stored(report_hash)
    .call();
  console.log(
    "未病测评报告时是否存储:",
    `${result}`
  );
};

//5.查询某个手表是否存储在合约中

const get_watch_store_status = async (
  imei: string
) => {
  let result = await reward.methods
    .watch_is_stored(imei)
    .call();

  console.log("脉诊手表是否存储:", `${result}`);
};

//6.存储报告哈希

const r_hash = nanoid();

const store_reportHash = async (
  report_hash: string
) => {
  console.log(
    "call store_reportHash function......"
  );

  const address =
    "0xFb7032b3fcfFc0A41E96B99AFd663A477819667C";

  //构建交易对象
  const tx: any = {
    from: address,
    to: contract_address,
    gas: 50000,
    data: reward.methods
      .store_reportHash(report_hash)
      .encodeABI(),
  };

  //签名
  const signed: any =
    await web3.eth.accounts.signTransaction(
      tx,
      PRIVATE_KEY.private_key
    );

  console.log(
    "Transaction process with hash",
    signed
  );

  web3.eth
    .sendSignedTransaction(signed.rawTransaction)
    .on("receipt", (receipt) => {
      console.log("store_hash receipt", receipt);
      console.log("store_hash finished");
    });
};

//7.存储报告哈希

const get_reward_amount = async () => {
  let result = await reward.methods
    .reward_amount()
    .call();

  console.log(
    "当前提现金额:",
    Big(result)
      .div(10 ** 18)
      .toFixed(0)
  );
};

const get_next_reward_time = async (
  imei: string
) => {
  let result = await reward.methods
    .get_next_reward_time(imei)
    .call();
  console.log("6.下次提现时间:", `${result}`);
};

//7 存储手表

const imei = nanoid();

const store_watch = async (imei: string) => {
  console.log("call store_watch function......");

  const address =
    "0xFb7032b3fcfFc0A41E96B99AFd663A477819667C";

  //构建交易对象
  const tx: any = {
    from: address,
    to: contract_address,
    gas: 5000000,
    data: reward.methods
      .store_watch(imei)
      .encodeABI(),
  };

  //签名
  const signed: any =
    await web3.eth.accounts.signTransaction(
      tx,
      PRIVATE_KEY.private_key
    );

  console.log(
    "Transaction process with hash:",
    signed
  );

  web3.eth
    .sendSignedTransaction(signed.rawTransaction)
    .on("receipt", (receipt) => {
      console.log("store_watch receipt", receipt);
      console.log("%s stored:", imei);
    });

  return imei;
};

console.log("获取到的随机IMEI码:", imei);

// 转账

const set_reward_amount = async (
  amount: number
) => {
  console.log("call set amount function......");

  const address =
    "0xFb7032b3fcfFc0A41E96B99AFd663A477819667C";

  //构建交易对象
  const tx: any = {
    from: address,
    to: contract_address,
    gas: 5000000,
    data: reward.methods
      .set_reward_amount(amount)
      .encodeABI(),
  };

  //签名
  const signed: any =
    await web3.eth.accounts.signTransaction(
      tx,
      PRIVATE_KEY.private_key
    );

  console.log(
    "Transaction process with hash:",
    signed
  );

  web3.eth
    .sendSignedTransaction(signed.rawTransaction)
    .on("receipt", (receipt) => {
      console.log(
        "set reward amount receipt",
        receipt
      );
      console.log("set_reward_amount finished");
    });
};

const to: any =
  "0xec876F62798E65270Ef163f86ec7afE5E7D634e7";

const transfer_erc20 = async (
  token_contract_address: string,
  watch_id: string,
  report_hash: string,
  receive_address: string
) => {
  console.log("call set amount function......");

  const address =
    "0xFb7032b3fcfFc0A41E96B99AFd663A477819667C";

  //构建交易对象
  const tx: any = {
    from: address,
    to: contract_address,
    gas: 5000000,
    data: reward.methods
      .transfer_erc20(
        token_contract_address,
        watch_id,
        report_hash,
        receive_address
      )
      .encodeABI(),
  };

  //签名
  const signed: any =
    await web3.eth.accounts.signTransaction(
      tx,
      PRIVATE_KEY.private_key
    );

  console.log(
    "Transaction process with hash:",
    signed
  );

  web3.eth
    .sendSignedTransaction(signed.rawTransaction)
    .on("receipt", (receipt) => {
      console.log(
        "set reward amount receipt",
        receipt
      );
      console.log("set_reward_amount finished");
    });
};

const amount: number = 100000;
const set_amount = Big(amount)
  .times(10 ** 18)
  .toFixed(0);

// store_watch("watch3");
// store_reportHash("hash3");
// set_reward_amount(set_amount);
// transfer_erc20(token_address,"watch3","hash3",to);

// get_reward_amount();
// get_watch_store_status("watch3");
// get_report_hash_store_status("hash3");
get_next_reward_time("watch3");

// const moonbeam_web3 = new Web3(
//   "wss://wss.api.moonbase.moonbeam.network"
// );

// moonbeam_web3.eth
//   .subscribe(
//     "newBlockHeaders",
//     (error, result) => {
//       if (error) console.error(error);
//     }
//   )
//   .on("connected", function (subscriptionId) {
//     console.log(subscriptionId);
//   })
//   .on("data", function (log) {
//     console.log(log);
//   });
```

# 6 事件处理

# 7 Hooks

使得函数组件更强大、更灵活的钩子函数（在某一个情况下自动执行）

函数组件更加适合React的设计理念：UI = f (data)

# 8 路由



