# Telegram Bot 使用文档

- [文档原文](https://www.cnblogs.com/kainhuck/p/13576012.html)
- [官方文档](https://core.telegram.org/bots/api)
- [使用 Cloudflare Workers 反代API](./telegram-bot-api.md)

## 创建机器人

在telegram中我们可以通过和一个名为`BotFather`的机器人交互来申请我们自己的机器人，具体步骤如下

1. 添加BotFather为好友

   [点击这里](https://telegram.me/botfather)添加BotFather

2. 打开和BotFather的对话框发送 `/newbot`

   这一步过后BotFather会提示你输入你要创建的机器人的名字，这个名字可以随意，是我们称呼它的名字

3. 设置自定义机器人的名字（这个名字不同于上一步的名字，这个名字是唯一的）结尾必须是`_bot`或者`Bot`，不能包含中文, 标点符号

4. 如果上一步执行成功那么BotFather会返回该机器人的token，大概长这样

   ```
   123456789:ABCDEfghiJK4314daDSadSa7
   ```

   记住这个token,到这里机器人就创建好了

## 将机器人添加到群组里

进入机器人信息页面，点击`更多`，点击`添加到群组`，选择一个群组即可

## 获取群组chat_id

通常来说我们都需要让机器人在一个群组里工作，所以首先我们需要将机器人添加到我们指定的群组，在群组里发送随意消息并@这个机器人，比如

```
hello @your_bot
```

然后浏览器打开这个链接，注意替换为你的token

```
https://api.telegram.org/bot<token>/getUpdates
```

你看到的是一个json，格式如下

```json
{
  "ok": true,
  "result": [
    {
      "update_id": 414941268,
      "message": {
        ...
      },
      "chat": {
        "id": -465512321,
        ...
      },
      ...
    }
  ]
}
```

从中找到chat.id这就是当前群组的id,以后发消息都是发到这个id.

## 机器人发送请求

### 请求接口

telegram发送消息的方式类似与钉钉机器人，都是向一个api发送http请求，而且对于同一个API`telegram`支持`GET`和`POST`两种请求方式.请求的api格式如下

```
https://api.telegram.org/bot<token>/<method>
```

其中token为你的机器人token，method为telegram给定的方法，在获取群组chat_id那一步就使用了telegram的其中一个方法(getUpdates),其他方法后面会介绍

### 携带参数

请求api时有些方法需要携带参数，telegram支持的传参方式/类型如下

- URL查询参数
- `application/x-www-form-urlencoded`
- `application/json`
- `multipart/form-data` (上传文件使用这个content-type)

### 获取响应

对于每次请求telegram都会有一个响应，响应的内容是一个json，格式如下

```json
{
  "ok": true,
  "result": ...
}
```

其中返回的result可以是telegram定义的对象或者是对象的列表

## telegram对象

telegram机器人的几乎所有操作都是一个一个对象的操作。

telegram定义了许多场景下的对象，[详见](https://core.telegram.org/bots/api#available-types)，这里举例一些常见的

### Update

该对象表示传入的更新，比如接收到用户发来的新消息，就会获得新的更新

| Field                | Type                                                         | Description                                                  |
| :------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| update_id            | Integer                                                      | 更新的唯一标识符。 更新标识符从某个正数开始，并依次增加。 如果您使用的是Webhooks，则此ID变得尤为方便，因为它使您可以忽略重复的更新或在错误的情况下恢复正确的更新顺序。 如果至少有一个星期没有新更新，则将随机选择下一个更新的标识符，而不是顺序选择。 |
| message              | [Message](https://core.telegram.org/bots/api#message)        | 可选的。任何形式的新传入消息-文本，照片，贴纸等。            |
| edited_message       | [Message](https://core.telegram.org/bots/api#message)        | 可选的。机器人已知并已编辑的消息的新版本                     |
| channel_post         | [Message](https://core.telegram.org/bots/api#message)        | 可选的。任何形式的新传入频道帖子-文字，照片，贴纸等。        |
| edited_channel_post  | [Message](https://core.telegram.org/bots/api#message)        | 可选的。机器人已知并已编辑的频道发布的新版本                 |
| inline_query         | [InlineQuery](https://core.telegram.org/bots/api#inlinequery) | 可选的。新的传入内联查询                                     |
| chosen_inline_result | [ChosenInlineResult](https://core.telegram.org/bots/api#choseninlineresult) | 可选的。用户选择并发送给其聊天伙伴的内联查询的结果。请参阅收集反馈的文档，以获取有关如何为您的机器人启用这些更新的详细信息。 |
| callback_query       | [CallbackQuery](https://core.telegram.org/bots/api#callbackquery) | 可选的。新传入的回调查询                                     |
| shipping_query       | [ShippingQuery](https://core.telegram.org/bots/api#shippingquery) | 可选的。新的收货查询。仅适用于价格灵活的发票                 |
| pre_checkout_query   | [PreCheckoutQuery](https://core.telegram.org/bots/api#precheckoutquery) | 可选的。新的传入预结帐查询。包含有关结帐的完整信息           |
| poll                 | [Poll](https://core.telegram.org/bots/api#poll)              | 可选的。新的投票状态。机器人仅接收有关僵尸程序发送的有关已停止的投票和投票的更新 |
| poll_answer          | [PollAnswer](https://core.telegram.org/bots/api#poll_answer) | 可选的。用户在非匿名调查中更改了答案。僵尸程序仅在由僵尸程序本身发送的民意调查中才能获得新的选票。 |

任何给定的更新中最多只能存在一个可选参数。

### User

该对象表示telegram的一个用户或者机器人

| Field                      | Type    | Description                                                  |
| :------------------------- | :------ | :----------------------------------------------------------- |
| id                         | Integer | 该用户或机器人的唯一标识                                     |
| is_bot                     | Boolean | 标识该用户是否是机器人，True如果是机器人                     |
| first_name                 | String  | 用户或者机器人的first_name                                   |
| last_name                  | String  | 可选。用户或者机器人的last_name                              |
| username                   | String  | 可选。用户或者机器人的username                               |
| language_code              | String  | 可选。用户语言的IETF语言标签                                 |
| can_join_groups            | Boolean | 可选。返回True如果该机器人可以被邀请加入群组，只在`getMe`方法返回 |
| can_read_all_group_message | Boolean | 可选。返回True如果该机器人禁用了隐私模式，只在`getMe`方法返回 |
| supports_inline_queries    | Boolean | 可选。返回True，如果这个自持内联查询，只在`getMe`方法返回    |

### Chat

该对象表示一个聊天信息

| Field               | Type                                                         | Description                                                  |
| :------------------ | :----------------------------------------------------------- | :----------------------------------------------------------- |
| id                  | Integer                                                      | 该聊天的唯一标识符。 这个数字可能会大于32位但是一定小于52位所以编程时因指定`int64`类型 |
| type                | String                                                       | 聊天的类型，可以是 “private”, “group”, “supergroup” 或者 “channel” |
| title               | String                                                       | 可选。 标题, 针对 supergroups, channels 和 group 类型的聊天  |
| username            | String                                                       | 可选。 Username, 针对 私有的聊天，如果可以的话也针对 supergroups 和 channels |
| first_name          | String                                                       | 可选。 私人聊天中对方的first_name                            |
| last_name           | String                                                       | 可选。私人聊天中对方的last_name                              |
| photo               | [ChatPhoto](https://core.telegram.org/bots/api#chatphoto)    | 可选。 聊天照片，只在`getChat`方法返回                       |
| description         | String                                                       | 可选。 描述, 针对 groups, supergroups 和 channel 的聊天。只在`getChat`方法返回 |
| invite_link         | String                                                       | 可选。聊天的邀请链接, 针对 groups, supergroups 和 channel 的聊天。聊天中的每个管理员都会生成自己的邀请链接，因此机器人必须首先使用`exportChatInviteLink`生成链接。仅在`getChat`中返回。 |
| pinned_message      | [Message](https://core.telegram.org/bots/api#message)        | 可选。固定信息，针对 groups, supergroups 和 channels。只在`getChat`方法中返回 |
| permissions         | [ChatPermissions](https://core.telegram.org/bots/api#chatpermissions) | 可选。默认的聊天成员权限, 针对 groups 和 supergroups。只在`getChat`方法中返回 |
| slow_mode_delay     | Integer                                                      | 可选。针对 supergroups, 每个非特权用户发送的连续消息之间允许的最小延迟。仅在`getChat`中返回。 |
| sticker_set_name    | String                                                       | 可选。针对 supergroups, 组贴纸集的名称。仅在getChat中返回。  |
| can_set_sticker_set | Boolean                                                      | 可选。 返回True如果机器人可以改变group的贴纸集，只在`getChat`方法中返回。 |

### Message

该对象代表一个消息

| Field                   | Type                                                         | Description                                                  |
| :---------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| message_id              | Integer                                                      | 此聊天中的唯一消息标识符                                     |
| from                    | [User](https://core.telegram.org/bots/api#user)              | 可选的。发件人，对于发送到channels的消息为空                 |
| date                    | Integer                                                      | 发送时间（Unix时间）                                         |
| chat                    | [Chat](https://core.telegram.org/bots/api#chat)              | 消息所属的会话                                               |
| forward_from            | [User](https://core.telegram.org/bots/api#user)              | 可选的。对于转发的消息，原始消息的发件人                     |
| forward_from_chat       | [Chat](https://core.telegram.org/bots/api#chat)              | 可选的。对于从频道转发的消息，有关原始频道的信息             |
| forward_from_message_id | Integer                                                      | 可选的。对于从频道转发的消息，是频道中原始消息的标识符       |
| forward_signature       | String                                                       | 可选的。对于从频道转发的消息，作者的签名（如果有）           |
| forward_sender_name     | String                                                       | 可选的。从用户转发的信息的发件人名称，这些用户不允许在转发的信息中添加指向其帐户的链接 |
| forward_date            | Integer                                                      | 可选的。对于转发的消息，原始消息的发送日期（Unix时间）       |
| reply_to_message        | [Message](https://core.telegram.org/bots/api#message)        | 可选的。对于答复，原始消息。请注意，即使此字段本身是答复，该字段中的Message对象也不会包含其他的reply_to_message字段。 |
| via_bot                 | [User](https://core.telegram.org/bots/api#user)              | 可选的。发送消息的机器人                                     |
| edit_date               | Integer                                                      | 可选的。消息最后一次编辑的日期（Unix时间）                   |
| media_group_id          | String                                                       | 可选的。该消息所属的媒体消息组的唯一标识符                   |
| author_signature        | String                                                       | 可选的。在频道中信息的作者签名                               |
| text                    | String                                                       | 可选的。对于文本消息，消息的实际UTF-8文本，0-4096个字符      |
| entities                | Array of [MessageEntity](https://core.telegram.org/bots/api#messageentity) | 可选的。对于文本消息，出现在文本中的特殊实体，例如用户名，URL，机器人命令等。 |
| animation               | [Animation](https://core.telegram.org/bots/api#animation)    | 可选的。消息是动画，有关动画的信息。为了向后兼容，设置此字段时，还将设置文档字段 |
| audio                   | [Audio](https://core.telegram.org/bots/api#audio)            | 可选的。消息是音频文件，有关该文件的信息                     |
| document                | [Document](https://core.telegram.org/bots/api#document)      | 可选的。消息是常规文件，有关文件的信息                       |
| photo                   | Array of [PhotoSize](https://core.telegram.org/bots/api#photosize) | 可选的。消息是照片，照片的可用尺寸                           |
| sticker                 | [Sticker](https://core.telegram.org/bots/api#sticker)        | 可选的。消息是贴纸，有关贴纸的信息                           |
| video                   | [Video](https://core.telegram.org/bots/api#video)            | 可选的。消息是视频，有关视频的信息                           |
| video_note              | [VideoNote](https://core.telegram.org/bots/api#videonote)    | 可选的。消息是视频注释，有关视频消息的信息                   |
| voice                   | [Voice](https://core.telegram.org/bots/api#voice)            | 可选的。消息是语音消息，有关文件的信息                       |
| caption                 | String                                                       | 可选的。动画，音频，文档，照片，视频或语音的标题，0-1024个字符 |
| caption_entities        | Array of [MessageEntity](https://core.telegram.org/bots/api#messageentity) | 可选的。对于带有标题的消息，出现在标题中的特殊实体，例如用户名，URL，漫机器人命令等 |
| contact                 | [Contact](https://core.telegram.org/bots/api#contact)        | 可选的。消息是共享的联系人，有关该联系人的信息               |
| dice                    | [Dice](https://core.telegram.org/bots/api#dice)              | 可选的。消息是一个骰子，具有从1到6的随机值                   |
| game                    | [Game](https://core.telegram.org/bots/api#game)              | 可选的。消息是一个游戏，有关游戏的信息。                     |
| poll                    | [Poll](https://core.telegram.org/bots/api#poll)              | 可选的。消息是原生投票，有关投票的信息                       |
| venue                   | [Venue](https://core.telegram.org/bots/api#venue)            | 可选的。消息是一个场地，有关该场地的信息。为了向后兼容，设置此字段时，还将设置位置字段 |
| location                | [Location](https://core.telegram.org/bots/api#location)      | 可选的。消息是共享位置，有关位置的信息                       |
| new_chat_members        | Array of [User](https://core.telegram.org/bots/api#user)     | 可选的。添加到组或超组中的新成员以及有关它们的信息（机器人本身可能是这些成员之一） |
| left_chat_member        | [User](https://core.telegram.org/bots/api#user)              | 可选的。成员已从群组中删除，有关他们的信息（该成员可能是机器人本身） |
| new_chat_title          | String                                                       | 可选的。聊天标题已更改为此值                                 |
| new_chat_photo          | Array of [PhotoSize](https://core.telegram.org/bots/api#photosize) | 可选的。聊天照片已更改为此值                                 |
| delete_chat_photo       | True                                                         | 可选的。服务消息：聊天照片已删除                             |
| group_chat_created      | True                                                         | 可选的。服务信息：组已创建                                   |
| supergroup_chat_created | True                                                         | 可选的。 服务消息：超组已创建。 在通过更新发送的消息中无法接收到该字段，因为bot在创建时不能成为超组的成员。 仅当有人回复直接创建的超组中的第一条消息时，才可以在reply_to_message中找到该消息。 |
| channel_chat_created    | True                                                         | 可选的。 服务信息：频道已创建。 在通过更新发送的消息中无法接收到该字段，因为bot在创建时不能成为频道的成员。 如果有人回复频道中的第一条消息，则只能在reply_to_message中找到它。 |
| migrate_to_chat_id      | Integer                                                      | 可选的。 该组已迁移到具有指定标识符的超组。 此数字可能大于32位，并且某些编程语言在解释它时可能会有困难/无声的缺陷。 但是它小于52位，因此带符号的64位整数或双精度浮点类型对于存储此标识符是安全的。 |
| migrate_from_chat_id    | Integer                                                      | 可选的。 超级组已从具有指定标识的组中迁移。 此数字可能大于32位，并且某些编程语言在解释它时可能会有困难/无声的缺陷。 但是它小于52位，因此带符号的64位整数或双精度浮点类型对于存储此标识符是安全的。 |
| pinned_message          | [Message](https://core.telegram.org/bots/api#message)        | 可选的。 指定的消息已固定。 请注意，即使该字段本身是答复，该字段中的Message对象也不会包含其他的reply_to_message字段。 |
| invoice                 | [Invoice](https://core.telegram.org/bots/api#invoice)        | 可选的。 消息是付款的发票，有关发票的信息。                  |
| successful_payment      | [SuccessfulPayment](https://core.telegram.org/bots/api#successfulpayment) | 可选的。 消息是有关成功付款的服务消息，有关付款的信息。      |
| connected_website       | String                                                       | 可选的。 用户登录的网站的域名。                              |
| passport_data           | [PassportData](https://core.telegram.org/bots/api#passportdata) | 可选的。 电报护照数据                                        |
| reply_markup            | [InlineKeyboardMarkup](https://core.telegram.org/bots/api#inlinekeyboardmarkup) | 可选的。 消息附带的嵌入式键盘。 login_url按钮表示为普通url按钮。 |

### MessageEntity

该对象表示文本消息中的一个特殊实体。例如，标签，用户名，URL等。

| Field    | Type                                            | Description                                                  |
| :------- | :---------------------------------------------- | :----------------------------------------------------------- |
| type     | String                                          | 实体的类型。 可以是“mention”（@username），<br>“hashtag”（#hashtag），<br>“cashtag”（$ USD），<br>“ bot_command”（/ start @ jobs_bot），<br>“ URL”（[https://telegram.org](https://telegram.org/)），<br>“ email”（do-not-reply@telegram.org），<br>“phone_number”（+ 1-212-555-0123），<br>“bold”（粗体），“italic”（斜体），<br>“underline”（带下划线的文本） ），“strikethrough”（删除线文本），<br>“code”（等宽字符串），“ pre”（等宽块），<br>“ text_link”（对于可点击的文本URL），<br>“ text_mention”（对于没有用户名的用户） |
| offset   | Integer                                         | 以UTF-16代码单位向实体开始的偏移量                           |
| length   | Integer                                         | 实体的长度（以UTF-16代码单元为单位）                         |
| url      | String                                          | 可选的。仅对于“ text_link”，用户点击文本后将打开的URL        |
| user     | [User](https://core.telegram.org/bots/api#user) | 可选的。仅针对“ text_mention”，提到的用户                    |
| language | String                                          | 可选的。仅对于“ pre”，实体文本的编程语言                     |

### BotCommand

这个对象代表值一条机器人指令

| Field       | Type   | Description                                                |
| :---------- | :----- | :--------------------------------------------------------- |
| command     | String | 命令文本，1-32个字符。只能包含小写英文字母，数字和下划线。 |
| description | String | 命令说明，3-256个字符。                                    |

### WebhookInfo

这个对象表示当前webhook的状态

| Field                  | Type            | Description                                                  |
| :--------------------- | :-------------- | :----------------------------------------------------------- |
| url                    | String          | Webhook URL，如果未设置webhook，则可能为空                   |
| has_custom_certificate | Boolean         | 如果为webhook证书检查提供了自定义证书则为真                  |
| pending_update_count   | Integer         | 等待交付的更新数量                                           |
| last_error_date        | Integer         | 可选的。尝试通过Webhook传递更新时发生的最新错误的Unix时间    |
| last_error_message     | String          | 可选的。尝试通过Webhook传递更新时发生的最新错误的人类可读格式的错误消息 |
| max_connections        | Integer         | 可选的。与Webhook进行更新交付的同时HTTPS连接的最大允许数量   |
| allowed_updates        | Array of String | 可选的。机器人已订阅的更新类型的列表。默认为所有更新类型     |

### ReplyKeyboardMarkup

该对象表示带有回复选项的自定义键盘

| Field             | Type                                                         | Description                                                  |
| :---------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| keyboard          | Array of Array of [KeyboardButton](https://core.telegram.org/bots/api/#keyboardbutton) | 按钮行数组，每个行由一个KeyboardButton对象数组表示           |
| resize_keyboard   | Boolean                                                      | 可选的。请求客户垂直调整键盘大小以达到最佳适合度（例如，如果只有两行按钮，则使键盘变小）。默认为false，在这种情况下，自定义键盘的高度始终与应用程序的标准键盘相同。 |
| one_time_keyboard | Boolean                                                      | 可选的。要求客户在使用键盘后立即隐藏它。键盘仍然可用，但是客户端将在聊天中自动显示常用的字母键盘-用户可以在输入字段中按特殊按钮以再次查看自定义键盘。默认为false。 |
| selective         | Boolean                                                      | 可选的。如果只想向特定用户显示键盘，请使用此参数。目标：1）在Message对象的文本中@提及的用户； 2）如果机器人的消息是回复（具有reply_to_message_id），则为原始消息的发送者。 示例：用户请求更改机器人的语言，机器人用键盘答复选择新语言的请求。群组中的其他用户看不到键盘。 |

### KeyboardButton

该对象表示回复键盘的一个按钮。对于简单的文本按钮，可以使用`String`代替此对象来指定按钮的文本。可选字段`request_contact`，`request_location`和`request_poll`是互斥的。

| Field            | Type                                                         | Description                                                  |
| :--------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| text             | String                                                       | 按钮的文字。如果未使用任何可选字段，则在按下按钮时它将作为消息发送 |
| request_contact  | Boolean                                                      | 可选的。如果为True，则按下该按钮时，用户的电话号码将作为联系人发送。仅在私人聊天中可用 |
| request_location | Boolean                                                      | 可选的。如果为True，则在按下按钮时将发送用户的当前位置。仅在私人聊天中可用 |
| request_poll     | [KeyboardButtonPollType](https://core.telegram.org/bots/api/#keyboardbuttonpolltype) | 可选的。如果指定，则将要求用户创建一个民意调查，并在按下按钮时将其发送给机器人。仅在私人聊天中可用 |

**注意**：*request_contact*和*request_location*选项仅适用于2016年4月9日之后发布的电报版本。较旧的客户端将显示不受支持的消息。

**注意**：*request_poll*选项仅在2020年1月23日之后发布的电报版本中有效。旧客户端将显示不支持的消息。

### KeyboardButtonPollType

该对象表示民意调查的类型，可以在按下相应按钮时创建并发送该民意调查

| Field | Type   | Description                                                  |
| :---- | :----- | :----------------------------------------------------------- |
| type  | String | 可选的。如果通过了测验，将仅允许用户以测验模式创建民意测验。如果通过常规，则仅允许常规民意调查。否则，将允许用户创建任何类型的民意测验。 |

### ReplyKeyboardRemove

收到带有此对象的消息后，Telegram客户端将删除当前的自定义键盘并显示默认的字母键盘。默认情况下，将显示自定义键盘，直到机器人发送新键盘为止。一次性键盘的例外情况是用户按下按钮后立即隐藏的一次性键盘

| Field           | Type    | Description                                                  |
| :-------------- | :------ | :----------------------------------------------------------- |
| remove_keyboard | True    | 请求客户端删除自定义键盘（用户将无法召唤此键盘；如果要隐藏键盘，但保持其可访问性，请在ReplyKeyboardMarkup中使用one_time_keyboard） |
| selective       | Boolean | 可选的。如果仅要为特定用户卸下键盘，请使用此参数。目标：1）在Message对象的文本中@提及的用户； 2）如果漫游器的消息是回复（具有reply_to_message_id），则为原始消息的发送者。 示例：用户在投票中投票，机器人返回确认消息以回应投票，并删除该用户的键盘，同时仍向尚未投票的用户显示带有投票选项的键盘。 |

### InlineKeyboardMarkup

该对象表示一个嵌入式键盘，出现在其所属消息的旁边。

| Field           | Type                                                         | Description                                              |
| :-------------- | :----------------------------------------------------------- | :------------------------------------------------------- |
| inline_keyboard | Array of Array of [InlineKeyboardButton](https://core.telegram.org/bots/api/#inlinekeyboardbutton) | 按钮行数组，每个行由一个InlineKeyboardButton对象数组表示 |

**注意**：这仅适用于2016年4月9日之后发布的电报版本。较旧的客户端将显示不受支持的消息。

### InlineKeyboardButton

此对象表示嵌入式键盘的一个按钮。您必须完全使用可选字段之一。

| Field                            | Type                                                         | Description                                                  |
| :------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| text                             | String                                                       | 在按钮上标记文本                                             |
| url                              | String                                                       | 可选的。按下按钮时将打开HTTP或tg:// URL                      |
| login_url                        | [LoginUrl](https://core.telegram.org/bots/api/#loginurl)     | 可选的。用于自动授权用户的HTTP URL。可以替代电报登录小部件。 |
| callback_data                    | String                                                       | 可选的。按下按钮时要在回调查询中发送到bot的数据，1-64个字节  |
| switch_inline_query              | String                                                       | 可选的。如果已设置，则按下按钮将提示用户选择其聊天之一，打开该聊天并将bot的用户名和指定的内联查询插入输入字段。可以为空，在这种情况下，只会插入机器人的用户名。 <br>**注意**：这为用户提供了一种简便的方法，使他们在当前与它进行私聊时以内联模式开始使用您的机器人。与switch_pm…操作结合使用时特别有用–在这种情况下，用户将自动返回到其切换到的聊天中，而跳过聊天选择屏幕。 |
| switch_inline_query_current_chat | String                                                       | 可选的。如果已设置，则按下按钮会将bot的用户名和指定的嵌入式查询插入当前聊天的输入字段中。可以为空，在这种情况下，只会插入机器人的用户名。 |
| callback_game                    | [CallbackGame](https://core.telegram.org/bots/api/#callbackgame) | 可选的。用户按下按钮时将启动的游戏的描述。 **注意**：此类型的按钮必须始终是第一行中的第一个按钮。 |
| pay                              | Boolean                                                      | 可选的。指定True，发送付款按钮。  **注意**：此类型的按钮必须始终是第一行中的第一个按钮。 |

### ForceReply

收到带有该对象的消息后，Telegram客户端将向用户显示一个答复界面（就像用户选择了机器人的消息并点按“答复”一样）。如果您要创建用户友好的逐步界面而不必牺牲隐私模式，这将非常有用。

| Field       | Type    | Description                                                  |
| :---------- | :------ | :----------------------------------------------------------- |
| force_reply | True    | 向用户显示回复界面，就像他们手动选择了机器人的消息并点按“回复”一样 |
| selective   | Boolean | 可选的。如果只想强制特定用户答复，请使用此参数。目标：1）在Message对象的文本中@提及的用户； 2）如果漫游器的消息是回复（具有reply_to_message_id），则为原始消息的发送者。 |

## telegram方法

telegram方法就是拼接在api后面的那串字符串，不区分大小写。[详见](https://core.telegram.org/bots/api#available-methods)，这里只列举一些常用的，下面我所指的返回是指返回json中的result部分，其他章节提到的所有方法均可以在这一章节查阅

### getUpdates

- 描述

  获取更新，还有另外一种获取更新的方法(webhook)，两种方式不能共存

- 参数

  无参数

- 返回

  `update`对象列表

### setWebhook

- 描述

  使用此方法可以指定URL并通过传出的Webhook接收传入的更新。只要机器人有更新，telegram就会向指定的URL发送HTTPS POST请求，请求数据为json序列化后的`update`对象

- 参数

  | Parameter       | Type                                                      | Required | Description                                                  |
    | :-------------- | :-------------------------------------------------------- | :------- | :----------------------------------------------------------- |
  | url             | String                                                    | Yes      | 发送更新的https url。使用空字符串删除webhook集成             |
  | certificate     | [InputFile](https://core.telegram.org/bots/api#inputfile) | Optional | 上传您的公共密钥证书，以便可以检查正在使用的根证书。有关详细信息，请参见我们的[自签名指南](https://core.telegram.org/bots/self-signed)。 |
  | max_connections | Integer                                                   | Optional | 与Webhook进行更新交付的同时HTTPS连接的最大允许数量为1-100。默认值为40。使用较低的值可以限制bot服务器的负载，使用较高的值可以增加bot的吞吐量。 |
  | allowed_updates | Array of String                                           | Optional | 您希望机器人接收的更新类型的JSON序列化列表。 例如，指定[“ message”，“ edited_channel_post”，“ callback_query”]仅接收这些类型的更新。 请参阅更新以获取可用更新类型的完整列表。 指定一个空列表以接收所有更新，无论类型如何（默认）。 如果未指定，将使用以前的设置。 请注意，此参数不会影响调用setWebhook之前创建的更新，因此可能会在短时间内收到不需要的更新。 |

- 返回

  成功返回`True`

### deleteWebhook

- 描述

  删除设置的webhook

- 参数

  无参数

- 返回

  成功返回True

### getWebhookInfo

- 描述

  获取当前webhook的状态

- 参数

  无参数

- 返回

  `WebhookInfo`对象

  （如果没有设置webhook，则返回的对象中url为空）

### getMe

- 描述

  获取机器人自身信息

- 参数

  无参数

- 返回

  `user`对象

### getChat

- 描述

  使用此方法可获取有关聊天的最新信息（一对一对话的用户的当前名称，用户的当前用户名，组或频道等）

- 参数

  | Parameter | Type              | Required | Description                                                  |
    | :-------- | :---------------- | :------- | :----------------------------------------------------------- |
  | chat_id   | Integer or String | Yes      | 目标聊天或目标超级组或频道的用户名的唯一标识符（格式为@channelusername） |

- 返回

  `chat`对象

### sendMessage

- 描述

  发送消息

- 参数

  | Parameter                | Type                                                         | Required | Description                                                  |
    | :----------------------- | :----------------------------------------------------------- | :------- | :----------------------------------------------------------- |
  | chat_id                  | Integer or String                                            | Yes      | 目标聊天（chat_id）或目标频道的用户名的唯一标识符（格式为@channelusername） |
  | text                     | String                                                       | Yes      | 待发送消息的文本，实体解析后为1-4096个字符                   |
  | parse_mode               | String                                                       | Optional | 消息文本中的实体解析模式。有关更多详细信息，请参见格式化选项。 |
  | disable_web_page_preview | Boolean                                                      | Optional | 禁用此消息中链接的链接预览                                   |
  | disable_notification     | Boolean                                                      | Optional | 静默发送消息。用户将收到没有声音的通知。                     |
  | reply_to_message_id      | Integer                                                      | Optional | 如果消息是答复，则为原始消息的ID                             |
  | reply_markup             | [InlineKeyboardMarkup](https://core.telegram.org/bots/api#inlinekeyboardmarkup) or [ReplyKeyboardMarkup](https://core.telegram.org/bots/api#replykeyboardmarkup) or [ReplyKeyboardRemove](https://core.telegram.org/bots/api#replykeyboardremove) or [ForceReply](https://core.telegram.org/bots/api#forcereply) | Optional | 其他界面选项。内联键盘，自定义回复键盘，删除回复键盘或强制用户回复的说明的JSON序列化对象。 |

- 返回

  刚刚发送的`message`对象

### setMyCommands

- 描述

  使用此方法可以更改机器人的命令列表

- 参数

  | Parameter | Type                                                         | Required | Description                                                  |
    | :-------- | :----------------------------------------------------------- | :------- | :----------------------------------------------------------- |
  | commands  | Array of [BotCommand](https://core.telegram.org/bots/api#botcommand) | Yes      | 将bot命令的JSON序列化列表设置为bot命令列表。最多可以指定100个命令。 |

- 返回

  成功返回`True`

### getMyCommands

- 描述

  使用此方法获取机器人命令的当前列表

- 参数

  无参数

- 返回

  `BotCommand`对象列表

-

## 格式化选项

格式化选项就是让我们的机器人以某种格式发送消息（比如markdown，或者html）

Bot API支持消息的基本格式。您可以在机器人的消息中使用粗体，斜体，下划线和删除线文本，以及内联链接和预格式化的代码。电报客户端将相应地呈现它们。您可以使用markdown样式或HTML样式格式。

请注意，Telegram客户端将在打开内联链接（“打开此链接？”以及完整的URL）之前向用户显示警报。

如果满足以下限制，则可以嵌套消息实体：

- 如果两个实体具有公共字符，则其中一个完全包含在另一个内部。
- 粗体，斜体，下划线和删除线实体可以包含并且要包含在任何其他实体中，但pre和code除外。
- 所有其他实体不能互相包含。

链接`tg://user?id=<user_id>`可以用于通过用户ID提及用户，而无需使用用户名。请注意：

- 这些链接仅在内联链接中使用时才有效。例如，当用于嵌入式键盘按钮或消息文本中时，它们将不起作用。
- 仅当用户过去联系过该机器人，通过内联按钮向该机器人发送了回调查询或成为提及该用户的组的成员时，才能保证这些提及有效。

### MarkdownV2 style

要使用此模式，请在parse_mode字段中传递MarkdownV2。在您的消息中使用以下语法：

~~~markdown
*bold \*text*
_italic \*text_
__underline__
~strikethrough~
*bold _italic bold ~italic bold strikethrough~ __underline italic bold___ bold*
[inline URL](http://www.example.com/)
[inline mention of a user](tg://user?id=123456789)
`inline fixed-width code`

```
pre-formatted fixed-width code block
```

```python
pre-formatted fixed-width code block written in the Python programming language
```
~~~

请注意：

- 任何代码在1到126之间（含1和126）的字符都可以在任何带有''字符的位置转义，在这种情况下，它将被视为普通字符，而不是标记的一部分。
- 在pre和code实体内部，所有'`'和'\'字符必须以前面的'\'字符转义。
- 内联链接定义的内部(...)部分，所有')'和'\'必须以前面的'\'字符转义
- 在其他所有地方这些字符 '_', '*', '[', ']', '(', ')', '~', '`', '>', '#', '+', '-', '=', '|', '{', '}', '.', '!' 必须用前置'\'转义
- 如果`斜体`和`下划线`之间存在歧义，`__`始终从左到右被视为`下划线`实体的开始或结尾，所以使用`___italic underline_\r__`代替`___italic underline___`

### HTML style

要使用此模式，请在parse_mode字段中传递HTML。当前支持以下标签：

```html
<b>bold</b>, <strong>bold</strong>
<i>italic</i>, <em>italic</em>
<u>underline</u>,
<ins>underline</ins>
<s>strikethrough</s>, <strike>strikethrough</strike>,
<del>strikethrough</del>
<b>bold <i>italic bold <s>italic bold strikethrough</s> <u>underline italic bold</u></i> bold</b>
<a href="http://www.example.com/">inline URL</a>
<a href="tg://user?id=123456789">inline mention of a user</a>
<code>inline fixed-width code</code>
<pre>pre-formatted fixed-width code block</pre>
<pre><code
        class="language-python">pre-formatted fixed-width code block written in the Python programming language</code></pre>
```

请注意：


- 当前仅支持上述标签。
- 所有不属于标记或HTML实体的`<`，`>`和`&`符号必须替换为相应的HTML实体(`<` 用 `&lt;`， `>` 用 `&gt;`， `&` 用 `&amp;`)。
- 支持所有数字HTML实体。
- 该API当前仅支持以下命名的HTML实体： `&lt;`, `&gt;`, `&amp;` and `&quot;`。
- 使用嵌套的pre和code标签，为pre实体定义编程语言。
- 不能为独立code标签指定编程语言。

### Markdown style

这是旧版模式，保留下来是为了向后兼容。要使用此模式，请在parse_mode字段中传递Markdown。在您的消息中使用以下语法：

~~~markdown
*bold text*
_italic text_
[inline URL](http://www.example.com/)
[inline mention of a user](tg://user?id=123456789)
`inline fixed-width code`

```
pre-formatted fixed-width code block
```

```python
pre-formatted fixed-width code block written in the Python programming language
```
~~~

请注意：

- 实体不得嵌套，而应使用解析模式`MarkdownV2`。
- 无法指定下划线和删除线实体，请改用解析模式`MarkdownV2`。
- 要在实体外部转义字符`_`，`*`，`，`[`，请在字符之前加上`\`。
- 不允许在实体内部转义，因此必须先关闭实体再重新打开：对于斜体使用`_snake_\__case_`，对于粗体`2*2=4`使用 `snake_case` 和 `*2*\**2=4*`。

## telegram更新

telegram中更新指的是机器人是否有收到新的消息，具体有哪些消息可以查看`telegram对象`部分中的`Update`，获取更新的方式有两种1. 轮询，2.webhook

### 轮询

这是一种主动询问的方式，这种方式比较简单但是效率欠佳，具体操作是，开发者每个一段时间请求一次`getUpdates`方法，从获取结果中判断update有无更新，有关`update`对象的描述可看`telegram对象`章节

### webhook

webhook可以理解为客户端给服务端的api,只要服务端一有更新就会主动将内容发送到客户端设置的一个api中，然后客户端收到消息后可做相应处理。

**给我们的机器人设置webhook**

通过`setWebhook`方法设置（前面有介绍），需要注意的是，telegram只支持`https`协议，所以我们的api服务器必须要有TLS证书，必须注意一但我们设置了webhook那么通过`getUpdates`方法将不起作用！

## telegram中的命令

通过命令来和机器人交互是电报机器人的一大特色，在telegram中命令由实体`BotCommand`表示（telegram对象那节已经介绍过）。

其实任何以`/`开头的连续英文消息都被视作命令(可以理解为这是一种具有格式的普通消息)，私聊或者在群里@机器人时发送`/`开头的信息可以查看实体类型为`bot_command`
。要想让机器人收到某条命令后执行响应动作只需要判断发送的消息是否匹配我们约定的内容即刻，（从某种意义上讲无论发送的消息是否是命令格式我们都可以将其视为命令只要我们愿意），开发具有命令响应的过程总结如下：

1. 起一个死循环监听`update`的`message`消息，可以是轮询或者webhook方式
2. 拿到`update.id`检查其是否更新，如有更新则取`message.Text`匹配已经定义好的`路由`（这是我定义的叫法，也就是我们约定的命令，事实上他的确和路由很像）
3. 如果匹配成功，执行我们定义的方法，如果匹配失败则当做普通信息无视，或者返回对应信息

从上面过程可以看出命令是否是`/`开头其实已经不那么重要了，那么为什么官方要定义`BotCommand`类呢，理由（优点）如下

1. 用以和普通消息区分
2. 在聊天消息中命令会高亮显示
3. 已经注册的命令在对话框中只需要输入`/`就会有提示列表

### 给机器人注册指令

**手动注册**

1. 和`BotFather`对话，输入`/mybot`

2. 选择要注册指令的机器人

3. 选择`Edit Bot`选项

4. 选择`Edit Commands`

5. 输入你想定义的命令，格式为

   ```
   command1 - 描述
   command2 - Description
   ```

   注意：注册时命令开头没有斜杠，使用命令时需要带上斜杠，中间用 `-` 分割；每一次的命令编辑都会覆盖之前的命令而不是追加，所以必须一次发送全部命令（在对话框中按`shift + enter`换行）

**通过`setMyCommands`方法注册**

- 详见（telegram方法 setMyCommands）

### 查看已注册的指令

- 详见（telegram方法 getMyCommands）

### 基本指令

telegram建议我们的机器人都带上三条基本指令分别是

- `/start`
- `/help`
- `/settings`

当设置了上面三个命令，用户首次打开与你的机器人的对话时，将看到`Start`按钮。机器人的个人资料页面上的菜单中将提供`Help`和`Settings`链接。

## 键盘

telegram中键盘也是机器人的一大特色，开发者可以自定义自己的键盘，一个键盘相当于机器人的菜单可以理解为一个答复界面，可以更加方便的和机器人交互。

telegram中的键盘有四种`ReplyKeyboardMarkup`，`InlineKeyboardMarkup`，`ReplyKeyboardRemove`和`ForceReply`，这四个对象可参考前面的介绍

### 创建键盘

只需要在`sendMessage`时指定`reply_markup`即可，详见`sendMessage`方法
