---
layout: post
title: "跟著 DHH 練習 ActionCable"
date: 2018-06-21 23:23:08 +0800
comments: true
categories: rails
---

<p>ActionCable 已經有一陣子了，剛好有時間來玩一下</p>

<!-- more -->

<p>主要跟著 DHH 的影片來練習一下，了解整個 ActionCable 的流程</p>

<ul>
<li><a href="https://www.youtube.com/watch?v=n0WUjGkDFS0">Rails 5: Action Cable demo</a></li>
</ul>

<h1 id="">介紹</h1>

<p>ActionCable 主要是一個 Pub/Sub 模型 + WebSocket 的 Ruby 框架，可以讓 Rails 實現即時通訊的方式</p>

<h3 id="websocket">WebSocket</h3>

<ul>
<li>是一種建立在單一 TCP 連線上的全雙工（full-duplex）通訊管道，可以讓網頁應用程式與伺服器之間做即時性、雙向的資料傳遞。</li>

<li>瀏覽器與伺服器之間若要建立一條 WebSocket 連線，在一開始的交握（handshake）階段中，要先從 HTTP 協定升級為 WebSocket 協定</li>
</ul>

<h3 id="polling">Polling 輪詢</h3>

<ul>
<li>瀏覽器每隔一段時間就自動送出一個 HTTP 請求給 server ，獲取最新的網頁資料</li>

<li>在 server 沒有新資料時，瀏覽器也是會自動送出請求，造成網路資源浪費</li>
</ul>

<h3 id="longpolling">Long-Polling 長時間輪詢</h3>

<ul>
<li>server 在接收到瀏覽器所送出的 HTTP 請求後， server 會等待一段時間，若在這段時間裡 server 有新的資料，它就會把最新的資料傳回給瀏覽器</li>

<li>如果等待的時間到了之後也沒有新資料的話，就會送一個回應給瀏覽器，告知瀏覽器資料沒有更新</li>

<li>如果在資料更新很頻繁的狀況下，長時間輪詢並不會比傳統的輪詢有效率，而且有時候資料量很大時，會造成連續的 polls 不斷產生，反而會更糟糕。</li>
</ul>

<h3 id="streaming">Streaming</h3>

<ul>
<li>讓 server 在接收到瀏覽器所送出 HTTP 請求後，立即產生一個回應瀏覽器的連線，並且讓這個連線持續一段時間不要中斷，而 server 在這段時間內如果有新的資料，就可以透過這個連線將資料馬上傳送給瀏覽器。</li>

<li>由於是建立在 HTTP 協定上的一種傳輸機制，所以有可能會因為代理 server（proxy）或防火牆（firewall）將其中的資料存放在緩衝區中，造成資料回應上的延遲，因此許多使用串流的 Comet 實作會在偵測到有代理 server 的狀況時，改用 Long-Polling 的方式處理。</li>
</ul>

<h3 id="pubsubpublishsubscribepattern">Pub/Sub 發佈/訂閱模式 (Publish/Subscribe Pattern)</h3>

<ul>
<li>發佈/訂閱設計模式是一種在即時通訊上很常用的架構，可以將通訊拆成發佈方和訂閱方，發佈方非同步地將訊息傳送給不定數量的訂閱方。</li>

<li>Pub/Sub 部分可以從你的主應用 Process 外，獨立出來成為一個單獨的運作元件。</li>
</ul>

<h1 id="actioncable">Actioncable 實作</h1>

<ul>
<li>建立新的專案</li>
</ul>

<pre><code class="ruby">rails new campfire
</code></pre>

<ul>
<li>建立 rooms controller</li>
</ul>

<pre><code class="ruby"># show 可以再產生 controller 自動產生 show action &amp; router
rails g controller rooms show
</code></pre>

<ul>
<li>修改 routes</li>
</ul>

<pre><code class="ruby"># config/routes.rb
Rails.application.routes.draw do
  root to: 'rooms#show'
end
</code></pre>

<ul>
<li>建立 message model</li>
</ul>

<pre><code class="ruby"># content:text 可自動在 migration 產生相對應的欄位
rails g model message content:text
rails db:migrate
</code></pre>

<ul>
<li>修改 rooms controller</li>
</ul>

<pre><code class="ruby"># controllers/rooms_controller.rb
class RoomsController &lt; ApplicationController
  def show
    @messages = Message.all
  end
end
</code></pre>

<pre><code class="ruby"># rails console 建立一筆來測試
Message.create!(content: "Hello World!")
</code></pre>

<ul>
<li>修改 rooms/show.html.erb 讓 messages 顯示</li>
</ul>

<pre><code class="ruby"># views/rooms/show.html.erb
&lt;h1&gt;Chat room&lt;/h1&gt;
&lt;div id="messages"&gt;
  &lt;%= render @messages %&gt;
&lt;/div&gt;
</code></pre>

<p>此時就可以 <code>rails s</code> 看一下首頁有沒有東西</p>

<ul>
<li>建立 channel</li>
</ul>

<p>speak 會自動產生 channel 的 action</p>

<p><code>rails g channel room speak</code></p>

<ul>
<li>Server 端 Mount ActionCable</li>
</ul>

<pre><code class="ruby"># config/routes.rb
Rails.application.routes.draw do
  root to: 'rooms#show'
  mount ActionCable.server =&gt; '/cable'
end
</code></pre>

<ul>
<li>修改 room.coffee</li>
</ul>

<p>請注意這邊是 coffee js 因此空格會有差別</p>

<pre><code class="ruby"># assets/javascripts/channels/room.coffee
App.room = App.cable.subscriptions.create "RoomChannel",
  connected: -&gt;
    # Called when the subscription is ready for use on the server

  disconnected: -&gt;
    # Called when the subscription has been terminated by the server

  received: (data) -&gt;
    # Called when there's incoming data on the websocket for this channel
    # 接到 data 後要做什麼處理，目前設定是將 message alert 彈出
    alert data['message']

  speak: (message)-&gt;
  # 傳 message 到 RoomChannel 的 speak action
    @perform 'speak', message: message
</code></pre>

<ul>
<li>修改 room channel 的 speak action</li>
</ul>

<pre><code class="ruby"># channels/room_channel.rb
class RoomChannel &lt; ApplicationCable::Channel
  def subscribed
    # 訂閱的頻道名稱
    stream_from "room_channel"
  end

  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
  end

  def speak(data)
    # 將 message 廣播到 room_channel 的 received function
    ActionCable.server.broadcast("room_channel", message: data['message'])
  end
end
</code></pre>

<p>重新啟動 server，在瀏覽器的 <code>console</code> 執行 </p>

<pre><code class="ruby"># 這個指令會 call client side 的 speak function，也就是 @perform 'speak', message: message
App.room.speak("Hello, World!") 
</code></pre>

<p>就會看到彈出視窗，表示伺服器端已經接受到訊息。</p>

<ul>
<li>新增 text input，讓 client 可以直接輸入 data</li>
</ul>

<pre><code class="ruby"># views/rooms/show.html.erb
&lt;h1&gt;Chat room&lt;/h1&gt;
&lt;div id="messages"&gt;
 &lt;%= render @messages %&gt;
&lt;/div&gt;
&lt;form&gt;
  &lt;label&gt;Say something:&lt;/label&gt;
  &lt;input type="text" data-behavior="room_speaker"&gt;
&lt;/form&gt;
</code></pre>

<ul>
<li>設定輸入後執行的動作</li>
</ul>

<p>記得先安裝 <code>jquery</code>，因為後面會利用 <code>jquery</code> 來操作 <code>dom</code> 和 <code>event</code></p>

<pre><code class="ruby"># Gemfile
gem 'jquery-rails'
</code></pre>

<pre><code class="ruby"># assets/javascripts/application.js
//= require jquery
</code></pre>

<p>請注意這邊是 coffee js 因此空格會有差別</p>

<pre><code class="ruby"># assets/javascripts/channels/room.coffee
App.room = App.cable.subscriptions.create "RoomChannel",
  connected: -&gt;
    # Called when the subscription is ready for use on the server

  disconnected: -&gt;
    # Called when the subscription has been terminated by the server

  received: (data) -&gt;
    # Called when there's incoming data on the websocket for this channel
    # 接到 data 後要做什麼處理，目前設定是將 message alert 彈出
    alert data['message']

  speak: (message)-&gt;
    @perform 'speak', message: message

# 設定在按下鍵盤 Enter 按鍵後，要執行 App.room.speak，就不用透過 console 去輸入
$(document).on 'keypress', '[data-behavior~=room_speaker]', (event) -&gt;
  if event.keyCode is 13 #return
    App.room.speak event.target.value
    event.target.value = ''
    event.preventDefault()
</code></pre>

<p>基本上現在就可以透過輸入框，讓 <code>alert</code> 跳出來了</p>

<h3 id="jobactioncable">透過 job 來執行 ActionCable</h3>

<ul>
<li>實際存取到 db</li>
</ul>

<pre><code class="ruby"># channels/room_channel.rb
class RoomChannel &lt; ApplicationCable::Channel
  def subscribed
    stream_from "room_channel"
  end

  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
  end

  def speak(data)
    # 改成直接 create record 在 db 裡面
    Message.create(content: data['message'])
  end
end
</code></pre>

<ul>
<li>create 後 callback 執行 Job</li>
</ul>

<pre><code class="ruby"># models/message.rb
class Message &lt; ApplicationRecord
  after_create_commit { MessageBroadcastJob.perform_later self }
end
</code></pre>

<ul>
<li>建立 job</li>
</ul>

<pre><code class="ruby">rails g job MessageBroadcast
</code></pre>

<pre><code class="ruby"># jobs/message_broadcast_job.rb
class MessageBroadcastJob &lt; ApplicationJob
  queue_as :default

  def perform(message)
     # perform_later 後會執行的動作
    ActionCable.server.broadcast("room_channel", message: render_message(message))
  end

  private

  def render_message(message)
     # 透過 ApplicationController 將 html code render 回來當一個參數做傳遞
    ApplicationController.renderer.render(partial: 'messages/message', locals: { message: message })
  end
end
</code></pre>

<p>此時 <code>rails s</code> 測試是否正常，因該會有 <code>alert</code> 跳出一段 <code>html code</code></p>

<ul>
<li>修改 received function</li>
</ul>

<pre><code class="ruby"># assets/javascripts/channels/room.coffee
App.room = App.cable.subscriptions.create "RoomChannel",
  connected: -&gt;
    # Called when the subscription is ready for use on the server

  disconnected: -&gt;
    # Called when the subscription has been terminated by the server

  received: (data) -&gt;
    # Called when there's incoming data on the websocket for this channel
    # alert 改成透過 jquery 找到 id='messages' 的 dom，並把回傳來的 html code append 上去
    $('#messages').append data['message']

  speak: (message)-&gt;
    @perform 'speak', message: message

$(document).on 'keypress', '[data-behavior~=room_speaker]', (event) -&gt;
  if event.keyCode is 13 #return
    App.room.speak event.target.value
    event.target.value = ''
    event.preventDefault()
</code></pre>

<p>再重啟一次，這時輸入資料後，畫面會立即新增，可以開雙畫面去做測試，另一邊的畫面是否也是立即新增</p>

<ul>
<li><a href="https://github.com/mgleon08/pratice_action_cable">完成範例程式碼</a></li>

<li><a href="http://mgleon08.github.io/blog/2017/10/06/http-websocket-bot/">之前玩過的 Http Websocket Bot</a></li>
</ul>

<p>參考文件:</p>

<ul>
<li><a href="https://www.youtube.com/watch?v=n0WUjGkDFS0">Rails 5: Action Cable demo</a></li>

<li><a href="https://ihower.tw/rails/actioncable.html">Action Cable 即時通訊</a></li>
</ul>