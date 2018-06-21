---
layout: post
title:  "Microservices using Rails, HTTP & RabbitMQ"
---

Imagine that, we have built one monolithic app **Cricket Live Score** where an
admin adds the live match scores into the database, and users use this app to get
to know the current score about the live match. Match like **India** v **Pakistan**
is very popular and it tends to have lots of traffic. Sometimes app goes down
because it can't handle such heavy traffic.

### Problem:
If you see here, there are two problems,

2. To get the current score, the user hits the page several times which causes
lots of incoming requests to the server and due to this app server goes down.
1. For a live match, if the server is down due to heavy load, then admin is not
able to do anything. He sits quietly.

It’s not a good reason for admin for sitting quietly because of heavy load on
the server. We have to handle admin part in a way that there is no effect of
users activities on it. There should not be a relation between them. Getting
current score by reloading the page is also not a right way if the app has such
traffic.

### Solution:
There are lots of solutions out there to handle this situation. But let’s see how
microservices sometimes play a crucial role in managing this situation.

Based on above two problems, let's create two microservices (i.e. two Rails apps).
In one microservice, we are going to handle only admin part (i.e. backend), and in
another microservice, we will handle user part (i.e. frontend). Building them
separately, we can scale them independently like handling more effective caching
in frontend microservice.

*NOTE: For frontend microservice, we can use other light-weight technologies
too like Sinatra instead of Rails.*

### Assumption:
In backend microservice, assume that we have added all required model, controller and
views files with code for match resource i.e. all the necessary business logic.
Admin user will come, add a new match and update related scores for the live
match. In frontend microservice, assume that we also have added a required
controller & views so that user can see the list of matches and can select any
one of them to see live scores of the match.

### Communication:
To speak within these microservices, we need a communication layer. For that,
either we can use **HTTP** or **RabbitMQ** (AMQP Protocol). But in this situation,
we are going to use both. Why both? Because, in frontend microservice, we need
data for matches and we will fetch it from backend microservice using **HTTP**
request. In backend service, when admin updates any match's live score, we also
need to update our users of frontend microservice in real-time. So users don't
need to refresh the page to get the current score of the match. For that,
we are going to use **RabbitMQ**.

### What is RabbitMQ?

RabbitMQ is Open Source messaging system sponsored by VMware.

> RabbitMQ is a message broker. The principal idea is pretty simple: it accepts and forwards messages. You can think about it as a post office: when you send mail to the post box you're pretty sure that Mr. Postman will eventually deliver the mail to your recipient. Using this metaphor RabbitMQ is a post box, a post office and a postman.

To access RabbitMQ from the browser we need this plugin **RabbitMQ Web-Stomp**
enabled.

RabbitMQ Web-Stomp plugin takes the [STOMP](http://stomp.github.io/) protocol and
exposes it using either plain WebSockets or
[a SockJS server](https://github.com/sockjs/sockjs-client).

Using these tools, we will push messages in real-time from RabbitMQ to
the web clients.

### Backend Microservice setup:

- Install [RabbitMQ](https://www.rabbitmq.com/download.html) and run it in
  background.
- Install [RabbitMQ Web-Stomp Plugin](http://www.rabbitmq.com/web-stomp.html) by
  enabling it `rabbitmq-plugins enable rabbitmq_web_stomp`.
- Add [bunny](https://github.com/ruby-amqp/bunny) gem to Gemfile
  `gem "bunny", ">= 2.5.1", require: false` and install it.

Restart RabbitMQ server and make sure that plugin is installed properly by
hitting this URL `http://127.0.0.1:15674/stomp`.

### Backend Microservice:

When admin updates the match score, we want to publish that score on the frontend
microservice.
First, create a file named `match_score_service.rb` under `lib` directory.
Make sure you autoload that file.

{% highlight ruby %}
require "bunny"

class MatchScoreService
  attr_reader :match

  def initialize(match_id)
    @match = Match.find_by id: match_id
  end

  def publish
    return if match.nil?

    exchange = channel.direct("match_scores")
    exchange.publish(payload, routing_key: queue.name, persistent: true)
    connection.close
  end

  private

  def payload
    match.to_json
  end

  def connection
    @conn ||= begin
                conn = Bunny.new
                conn.start
              end
  end

  def channel
    @channel ||= connection.create_channel
  end

  def queue
    @queue ||= channel.queue("match-#{match.id}", durable: true)
  end
end
{% endhighlight %}

Using Bunny Ruby client, we connect to RabbitMQ server on local machine. By
adding proper queue name and other settings, we send the match data to RabbitMQ
server. To know more about these settings/configurations, I would suggest following
this [tutorial](https://www.rabbitmq.com/tutorials/tutorial-one-ruby.html) on
RabbitMQ site.

Change your `matches_controller.rb` like this,

{% highlight ruby %}
class MatchesController < ApplicationController
  # ...
  def update
    respond_to do |format|
      if @match.update(match_params)
        MatchScoreService.new(@match.id).publish
        format.html { redirect_to edit_match_path(@match), notice: 'Match was successfully updated.' }
        format.json { render :show, status: :ok, location: @match }
      else
        format.html { render :edit }
        format.json { render json: @match.errors, status: :unprocessable_entity }
      end
    end
  end
  # ...
end
{% endhighlight %}

You will notice in `update` action, we handle publishing of match data.

### Frontend Microservice:
In backend microservice, we send match data in RabbitMQ server as a message.
Now we want to fetch that message in frontend microservice.

Before doing it, we also want to fetch matches data from backend microservice.
Let's do it by adding `http` gem to `Gemfile` and install it.

Now let's create a file named `crickinfo_service.rb` under `lib` folder which will
fetch matches lists and match object.

{% highlight ruby %}
class CrickinfoService
  def get_all_matches_info
    JSON.parse(Http.get("#{backend_espncrickinfo_url}/matches.json").body)["all_matches"]
  end

  def get_match_info(match_id)
    JSON.parse Http.get("#{backend_espncrickinfo_url}/matches/#{match_id}.json").body
  end

  private

  def backend_espncrickinfo_url
    "http://localhost:3000"
  end
end
{% endhighlight %}

Now in `matches_controller.rb`, add code like this,

{% highlight ruby %}
class MatchesController < ApplicationController
  def index
    service           = CrickinfoService.new
    all_matches       = service.get_all_matches_info
    @live_matches     = all_matches["live_matches"]
    @previous_matches = all_matches["previous_matches"]
  end

  def show
    service = CrickinfoService.new
    @match  = service.get_match_info(params[:id])
  end
end
{% endhighlight %}

In `index` action of `MatchesController`, we fetch matches list like `@live_matches`,
and `@previous_matches` from backend microservice and then we show them to users.
The user will pick any live match to get the real-time match updates. In `show`
action we fetch match's recent data then the user will start getting real-time
scores automatically if the match is live.

To get the real-time updates in `show.html.erb` page, we need to make a
connection to RabbitMQ server. Let's do it.

### Frontend Microservice setup:

To access RabbitMQ from browser side, download required files under
`app/assets/javascripts/` folder.

- [stomp.js](https://raw.githubusercontent.com/jmesnil/stomp-websocket/master/lib/stomp.js)
- [SockJS](https://github.com/sockjs/sockjs-client/blob/master/dist/sockjs.js)

Now in `show.html.erb` page, we want to handle accessing of RabbitMQ. To handle
it, please add code like this,

{% highlight html %}
  <script type="text/javascript">
    var ws = new SockJS("http://127.0.0.1:15674/stomp");
    var client = Stomp.over(ws);

    // RabbitMQ SockJS does not support heartbeats so disable them.
    client.heartbeat.outgoing = 0;
    client.heartbeat.incoming = 0;

    var onDebug = function(m) {
      console.log("DEBUG", m);
    };

    var onConnect = function() {
      console.log("Connected");
      var matchID   = $("[data-info~=match-title]").data('match-id');
      client.subscribe("/exchange/match_scores/match-" + matchID, function(data) {
        var match = JSON.parse(data.body);
        // Handle your code here & display real-time match score
        ...
        ...
        ...
      });
    };

    var onError = function(e) {
      console.log("ERROR", e);
    };

    client.debug = onDebug;
    client.connect("guest","guest", onConnect, onError,"/");
  </script>
{% endhighlight %}

If you see in above code, Stomp JavaScript client communicates to a STOMP
server over WebSocket. Once a STOMP client is created, it should call its
`connect()` to connect and authenticate to the STOMP server. In `connect()`
method we have passed default parameters `login` & `passcode` and those are
mandatory parameters. We also have passed callbacks so that we can fetch
the message from RabbitMQ server. You can read more about this flow in this
[article](http://jmesnil.net/stomp-websocket/doc/).

In `onConnect` method, we subscribe to queue and through which we get the
real-time message from RabbitMQ server if there is any. Once we get the match data
message, then we can handle our logic and show the match data to users in
the proper format.

This blog post is all about how we can decouple the app into separate components
using microservices which help to solve above two problems.

Happy Hacking :)

#### Complete source code:

1. Backend Microservice - [https://github.com/mechanicles/espn_cricinfo](https://github.com/mechanicles/espn_cricinfo)
2. Frontend Microservice - [https://github.com/mechanicles/espn_cricinfo_frontend](https://github.com/mechanicles/espn_cricinfo_frontend)

#### Demo:
1. Backend Microservice - [https://espn-cricinfo.herokuapp.com/](https://espn-cricinfo.herokuapp.com/)
2. Frontend Microservice - [ https://espn-cricinfo-frontend.herokuapp.com/]( https://espn-cricinfo-frontend.herokuapp.com/)

**NOTE: I don’t want to put anyone into the war of microservice app vs monolithic app,
here I’m only demonstrating the concept of the microservices.**
