#   Ace代码编辑器实践
官网:https://ace.c9.io/

在项目中使用ace编辑器，可实现代码编辑器，比如[腾讯云开发平台](https://studio.dev.tencent.com/)。

1. 引入

    ace的github上有一个仓库，是build好的各个版本ace代码，https://github.com/ajaxorg/ace-builds/。
    拉下来放在自己的项目上用就行。

    使用很简单，引入随便一个ace版本，我用的是src-min-noconflict版本，然后在一个初始化挂载在一个div上就行。

    ```javascript
        <template>
            <div ref="editor"></div> 
        </template>
        <script>
            import ace from 'ace-build-master'
            import 'ace-build-master/webpack-resolver'//webpack环境下引入
            import 'ace-build-master/src-min-noconflict/ext-language_tools'//设置自动补全需引入

            export default{
                data() {
                    return {
                        aceEditor:null,
                        fontSize:20,
                        theme:'ace/theme/twilight',
                        modePath: 'ace/mode/python',
                        value:'嘻嘻'
                    }
                }
                mounted() {
                    // 初始化编辑器，设置基本属性
                    this.aceEditor = ace.edit(this.$refs.editor, {
                        fontSize:this.fontSize,// 字体
                        theme:this.theme,//主题
                        mode:this.modePth//语言
                    })

                    // 设置自动补全
                    this.aceEditor.setOptions({
                        enableSnippets:true,
                        enableLiveAutocompletion:true,
                        enableBasicAutocompletion:true
                    })

                    // 初始化内容
                    this.aceEditor.setValue(`${this.value}`)
                    //其他还有很多设置网上一大堆
                }
            }
        </script>
       
    ```


2. 编辑器坑

    * websocket

    使用websocket通信，可以保证在打开多个网页的情况下，实时同步编辑器内容。
    １．使用websocket如果想带上cookie等请求头信息，需要保证请求头的origin和host相同（开发环境下需要设置一下）。
    ２．加入心跳机制、重连机制保证确定websock连接的死活。

    我用的是原生的websock：
    ```JavaScript
        const wsURL = `ws://xxxxxx`
        this.websocket = new WebSocket(wsURL)//如果没什么问题，这就连接上了
        // 下面是监听websock连接的四个状态，接收到信息、建立连接、错误、关闭
        this.websocket.onmessage = this.websocketonmessage
        this.websocket.onopen = this.websocketonopen
        this.websocket.onerror = this.websocketonerror
        this.websocket.onclose = this.websocketonclose


        websocketonmessage(e) {
            //建立连接后，开始心跳
            this.startHeartCheck()
        }

        websocketonerror(e) {
            //连接出错,重连
            this.reConnect()
        }
        websocketonclose(e) {
            //连接关闭，重连
            this.reConnect()
        }
        websocketonmessage(e) {
            //收到信息，充值心跳
            if(e.data === 'pong') {
                this.resetHeartCheck()
                return true
            }
        }

        startHeartCheck() {
           //心跳，发送ping给服务端，证明客户端还活着
           let self = this
           self.timeOutObj = setTimeout(function() {
               if(self.websock.read)
           })
        }
    ```

    

    