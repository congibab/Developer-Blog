# Unity_UdpSample

## 製作理由
他の学生さんにもUDP通信でネットワークゲームを開発が出来るように  
Sample Codeとクラスを制作しました。

## 実行環境
* Windows 10

## 使用ツール
* Visual Studio 2019
* Unity v2020.2.5f1

## 使用言語
* C# (System.Net.Sockets)
* Node.js v14.15.0 (dgram)

## 製作期間
* 2021/05/09 ~ 2021/05/11

## 具現した技能
* Client間のマッチング (2人制限)

## 動画(YouTube)
* [https://youtu.be/fKeTiaLIYA8](https://youtu.be/fKeTiaLIYA8)

## 製作人数
* 個人

## 製作者
* LEE GEONHWI(イゴンヒ)

# 使用方法

## Client
1. Assets/PrefabにNetworkManagerをSceneへ配置
2. NetworkManagerのInspector内にあるAddrass, Portを入力
3. 実行してConsole Logにエラーがなかったら成功
* 注意）サーバーを実行した後にClient(Unityt)を実行すること

## Server(Windows)
1. *ターミナル*を開く
2. cd *"ダウンロードした場所"/Unity_UdpSample/server/Sample2* 入力
3. *node index.js* 入力
4. server listening on *0.0.0.0:6060*が出力したら成功
5. 終了する時は、*Ctrl + C*を入力

# 説明(Client)

## NetworkManager.cs
* サーバーへの送受信を担当

```csharp
//サーバーと接続(サーバーIp, サーバーPort)
void Connect(string Adddrass, int Port);
//サーバへデータを送信
void Data_Send(JsonOBJ Massage);
```

```csharp
//evectType追加
public enum eventType
{
    init,
    chat,
    Message,
    //evectType追加可能
}
//==================================
//EventTypeによって処理分け
private void evectHandler(JsonOBJ data)
{
    eventType state = (eventType)Enum.Parse(typeof(eventType), data.type);

    switch (state)
    {
        case eventType.init :
            information.ribal_ip = data.ribal.address;
            information.ribal_port = data.ribal.port;
            break;

        case eventType.Message:
            break;

        //case evectType追加可能:
        //    break;
        
        default:
            break;
    }
}
```

## JsonOBJ.cs
* Json dataを各Typeで変換、管理
```csharp
[Serializable]
public class inform
{
    public string address;
    public int port;

    public string name;

    public Vector3 pos;
    public Vector3 rot;
    public Vector3 scal;
}
```

```csharp
[Serializable]
public class JsonOBJ
{
    public string type;
    public string msg;

    public inform own;
    public inform ribal;

    public JsonOBJ(eventType state)
    {
        this.type = state.ToString();
    }
}
```

## information.cs(カスタマイズ可能)
* サーバーから受けたAddressを保存
```csharp
public class information
{
    public static string own_ip { get; set; }
    public static int own_port { get; set; }
    public static string ribal_ip { get; set; }
    public static int ribal_port { get; set; }
}
```

# 説明(Server)

## index.js
* ClientへDataを送信する関数
``` js
function Emit(JsonObj, senderInfo) {

  var msg = JSON.stringify(JsonObj);
  server.send(msg, senderInfo.port, senderInfo.address, () => {
    //console.log(`Message sent to ${senderInfo.address}:${senderInfo.port} = ${msg}`)
  })
}
```

``` js
//=======================
//Jsonデータを解析してEvect処理
//=======================
function evectHandler(msg, senderInfo) {

  try {

    var data = JSON.parse(msg);

    console.log('Messages received ' + msg)
    var thisPlayerId = uuidv4();

    switch (data.type) {

      //マッチング処理
      case 'init':

        if (Object.keys(clients).length == 0) {
          setTimeout(clientsClear, 5000, clients);
        }

        for (let index in clients) {
          var JsonObj = data;
          JsonObj.ribal.address = clients[index].address;
          JsonObj.ribal.port = clients[index].port;
          Emit(JsonObj, senderInfo);
        }

        var thisPlayerId = uuidv4();
        clients[thisPlayerId] = senderInfo
        console.log(Object.keys(clients).length);
        console.log(senderInfo);


        var JsonObj = data;
        JsonObj.ribal.address = senderInfo.address;
        JsonObj.ribal.port = senderInfo.port;
        //JsonObj.ribal.pos.x = 0.5;
        Broadcast(JsonObj, clients, senderInfo);
        break;

        //case 'anoterEventType': 
        //break;

        default:

        break;
    }
  }

  catch (error) {
    console.log(error);
  }
}
```

## JsonInit
* Json構造
``` js
module.exports = class JsonInit{
    constructor () {
        this.jsonObj = {
            "type" : null,

            own : {
                "address" : null, 
                "port" : null,
                "uuid" : null,

                pos : {"x" : null, "y": null, "z": null},
                rot : {"x" : null, "y": null, "z": null}
            },

            ribal : {
                "address" : null, 
                "port" : null,
                "uuid" : null,

                pos : {"x" : null, "y": null, "z": null},
                rot : {"x" : null, "y": null, "z": null}
            }
        }
    }
    get() {
        return this.jsonObj;
    } 
}
```