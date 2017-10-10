connectNotes.md

Start 15.30

I forked the repo into my git.

Clone the repo from the command line:

```shell
git clone git@github.com:SimonLeigh/connect-eu-demo.git
```

I’m in a python virtualenv.

```shell
python --version
```
> Python 2.7.13

```shell 
which python
```
> /Users/simon/Documents/git/virtualenvs/couchbasetools/bin/python

```shell
./web-server.py 
```

>Traceback (most recent call last):
>  File "./web-server.py", line 7, in <module>
>    import tornado.gen
>ImportError: No module named tornado.gen

So I need to install tornado.

```shell
sudo -H  pip install tornado
```

Turns out I need to install twisted too

```shell
sudo -H pip install twisted
```

Try the web server again:

```shell
./web-server.py
```

>Traceback (most recent call last):
>...
> couchbase.exceptions._TimeoutError_0x17 (generated, catch TimeoutError): <RC=0x17[Client-Side timeout exceeded for operation. Inspect network conditions or increase the timeout], There was a problem while trying to send/receive your request over the network. This may be a result of a bad network or a misconfigured client or server, C Source=(src/bucket.c,785)>


So looks like it runs but times out because there is no server listening.

Now, I've got the vagrant repository already cloned so I just..

```shell
cd ~/Documents/git/vagrants/5.0.0-testing/ubuntu16
vagrant up
```

Of course I already have Virtualbox and vagrant installed...

Now I sit around because I don't have the ubuntu16 base image and it needs to download.

...and this breaks because something is wrong with puppet. Let's try ubuntu14 - vagrant box update

...slow downloads. 16:10 now.

OK, so created a 5.0.0 folder (because testing was so slow downloading from NAS) and added new IP rule to the top level vagrantfile.

I only have VAGRANT_NODES=1 so..

```shell
VAGRANT_NODES=4
```

This worked, now I have a cluster set up. So I'll hook the services up in 3 nodes.

Go to settings.py in the connect demo repo and change the IPs and bucket name.

Bucket name is now demo.

```shell
python create_dataset.py
```
>couchbase.exceptions._AuthError_0x2 (generated, catch AuthError): <RC=0x2[Authentication failed. You may have provided an invalid username/password combination], There was a problem while trying to send/receive your request over the network. This may be a result of a bad network or a misconfigured client or server, C Source=(src/bucket.c,785)>


Set up the user demo/password in Couchbase.

```shell
python create_dataset.py
```
> 

- **Returns nothing - we probably want to actually have this print something so success is clear.**

I check the cluster and find one production view, which is what we expect.
I have 26 documents in the bucket.

```shell
python web-server.py
```

>Traceback (most recent call last):
>  File "web-server.py", line 19, in <module>
>    tornado.platform.twisted.install()
>  File "/Users/simon/Documents/git/virtualenvs/couchbasetools/lib/python2.7/site-packages/tornado/platform/twisted.py", line 357, in install
>    installReactor(reactor)
>  File "/Users/simon/Documents/git/virtualenvs/couchbasetools/lib/python2.7/site-packages/twisted/internet/main.py", line 32, in installReactor
>    raise error.ReactorAlreadyInstalledError("reactor already installed")
> twisted.internet.error.ReactorAlreadyInstalledError: reactor already installed

Had a look at [this](https://groups.google.com/forum/#!topic/kivy-users/DNerfZRnCuc)

Changed the code and have pushed to git on my fork.

```
python web-server.py
```

So, if you then go to http://localhost:8888 - nothing happens.

http://localhost:8888/nodes.html does show the nodes correctly.

- **fixed this by changing the import order of Tornado**

Now it works. And it is 17:00.

Restart work @ 10:30

So I have now added some nodes back in whilst watching the /nodes endpoint and it works beautifully.

- **worth noting that we might want to suggest people don't add more data services when using VMs as the rebalance is so slow**

Next up, go to query workbench and create index

```sql
CREATE INDEX category ON demo(category)
```

Categories appeared successfully

- **might want to add to the script about running this what the expected behaviour is**

At this point I turned on autofailover with a 10s timeout (safety's sake on VMs)

Add search service. Happy days. It shows up in the node vis.

Create the English index. Search box pops up on website. Fantastic.

10:43 Download sync gateway, extract.

- **Could be much clearer here that one should edit sg config file for the setup**

```shell
./bin/sync_gateway ../android/sync_gateway_config.json
```

>2017-10-10T10:47:45.064+01:00 Enabling logging: [HTTP Auth]
>2017-10-10T10:47:45.065+01:00 ==== Couchbase Sync Gateway/1.5.0(494;5b6b5b9) ====
>2017-10-10T10:47:45.065+01:00 Configured process to allow 5000 open file descriptors
>2017-10-10T10:47:45.065+01:00 Opening db /charlie as bucket "demo", pool "default", server <http://10.142.174.104:8091>
>2017-10-10T10:47:45.066+01:00 GoCBCustomSGTranscoder Opening Couchbase database demo on <http://10.142.174.104:8091> as user "demo"
>2017-10-10T10:47:46.066+01:00 Using metadata purge interval of 3.00 days for tombstone compaction.
>_time=2017-10-10T10:47:46.085+01:00 _level=INFO _msg=Using plain authentication for user demo 
>_time=2017-10-10T10:47:46.085+01:00 _level=INFO _msg=Using plain authentication for user demo 
>_time=2017-10-10T10:47:46.085+01:00 _level=INFO _msg=Using plain authentication for user demo 
>2017-10-10T10:47:46.252+01:00 Sync fn rejected: new=map[order:[product:champagne product:red wine product:eggs product:tea bags product:marmite] _rev:1-55873a0acc58fdbbd62a8c5a4f118001 _id:Order::woop::2017-10-09T16:11:44.782278 type:order name:woop ts:1.507565504e+09]  old= --> 403 Invalid document type: order
>2017-10-10T10:47:46.252+01:00 WARNING: Unable to import doc "Order::woop::2017-10-09T16:11:44.782278" - external update will not be accessible via Sync Gateway.  Reason: 403 Invalid document type: order -- db.(*changeCache).DocChanged.func1() at change_cache.go:376
>2017-10-10T10:47:46.254+01:00 Sync fn rejected: new=map[_rev:1-45b799d1b37a1187141052b2a4d2d533 _id:Order::simon::2017-10-09T16:09:43.982250 type:order name:simon ts:1.507565383e+09 order:[product:crisps product:pot noodle product:marmite product:bananas product:burger]]  old= --> 403 Invalid document type: order
>2017-10-10T10:47:46.254+01:00 WARNING: Unable to import doc "Order::simon::2017-10-09T16:09:43.982250" - external update will not be accessible via Sync Gateway.  Reason: 403 Invalid document type: order -- db.(*changeCache).DocChanged.func1() at change_cache.go:376
>2017-10-10T10:47:46.265+01:00 Sync fn rejected: new=map[items:[product:burger product:ham product:sausages product:water product:champagne product:red wine product:beer product:cookie product:chocolate product:crisps product:cheese product:eggs product:bread product:butter product:milk product:bananas product:pineapple product:tea bags product:apples product:fish fingers product:pot noodle product:baked beans product:scotch egg product:marmite] _rev:1-9b7d2e4e1b6388bc66512bbdc9ead8ec _id:items]  old= --> 403 type is not provided.
>2017-10-10T10:47:46.265+01:00 WARNING: Unable to import doc "items" - external update will not be accessible via Sync Gateway.  Reason: 403 type is not provided. -- db.(*changeCache).DocChanged.func1() at change_cache.go:376
>2017-10-10T10:47:46.275+01:00 Sync fn rejected: new=map[_rev:1-524714a85ffd9e4a50508adf94d08e69 _id:Order::simon::2017-10-10T09:34:33.410875 type:order name:simon ts:1.507628073e+09 order:[product:baked beans product:tea bags product:pot noodle product:scotch egg product:fish fingers]]  old= --> 403 Invalid document type: order
>2017-10-10T10:47:46.275+01:00 WARNING: Unable to import doc "Order::simon::2017-10-10T09:34:33.410875" - external update will not be accessible via Sync Gateway.  Reason: 403 Invalid document type: order -- db.(*changeCache).DocChanged.func1() at change_cache.go:376
>2017-10-10T10:47:46.427+01:00 Sync fn rejected: new=map[ts:1.50762826e+09 order:[product:bananas product:bread product:ham product:sausages product:milk] _rev:1-aea8f69f0329d4ba66c2ad5438186f88 _id:Order::simon::2017-10-10T09:37:40.202970 type:order name:simon]  old= --> 403 Invalid document type: order
>2017-10-10T10:47:46.427+01:00 WARNING: Unable to import doc "Order::simon::2017-10-10T09:37:40.202970" - external update will not be accessible via Sync Gateway.  Reason: 403 Invalid document type: order -- db.(*changeCache).DocChanged.func1() at change_cache.go:376
>2017-10-10T10:47:46.722+01:00 Auth: Saved _sync:role:moderator: &{moderator  !:1,product-list.david.all_the_products:8 %!s(uint64=26)  %!s(*uint16=<nil>)}
>2017-10-10T10:47:46.722+01:00     Created role "moderator"
>2017-10-10T10:47:46.754+01:00 Auth: Saved _sync:role:admin: &{admin  !:1 %!s(uint64=27)  %!s(*uint16=<nil>)}
>2017-10-10T10:47:46.754+01:00     Created role "admin"
>2017-10-10T10:47:46.992+01:00 Auth: Saved _sync:user:mod: &{{mod  !:1 %!s(uint64=28)  %!s(*uint16=<nil>)} { %!s(bool=false) $2a$10$/9ct/ApHFRq.890.LvZLEOQ4iWr1Pu33a0jyXR40VUIqDE370uYca <nil> moderator:28  []} %!s(*auth.Authenticator=&{0xc42030a480 0xc42045e000}) []}
>2017-10-10T10:47:46.992+01:00     Created user "mod"
>2017-10-10T10:47:47.156+01:00 Auth: Saved _sync:user:admin: &{{admin  !:1 %!s(uint64=29)  %!s(*uint16=<nil>)} { %!s(bool=false) $2a$10$clM5Ut654yGdCdr9bEQgf.VL8YOAS.L9gLmJUlsVqITyZu6Wn4vei <nil> admin:29  []} %!s(*auth.Authenticator=&{0xc42030a480 0xc42045e000}) []}
>2017-10-10T10:47:47.156+01:00     Created user "admin"
>2017-10-10T10:47:47.303+01:00 Auth: Saved _sync:user:david: &{{david david:30  %!s(uint64=30)  %!s(*uint16=<nil>)} { %!s(bool=false) $2a$10$eZ3wULTT7cmLxrMoOEtAjO1wJiyC9AD0ubeDMjBsPUwEEw.TFkNey <nil>   []} %!s(*auth.Authenticator=&{0xc42030a480 0xc42045e000}) []}
>2017-10-10T10:47:47.304+01:00     Created user "david"
>2017-10-10T10:47:47.304+01:00 Starting admin server on 127.0.0.1:4985
>2017-10-10T10:47:47.308+01:00 Starting server on :4984 ... 


over to android studio... 10:51

Had to download AS 2.3.3 since I'm on 1.x. Office internet is fast so this only takes a minute but it is a 500mb download so ymmv.

10:55 Installed and away we go.

"open existing android studio project" and direct to the Android folder.

>Error:Failed to find target with hash string 'android-25' in: /Users/simon/Library/Android/sdk
><a href="install.android.platform">Install missing platform(s) and sync project</a>

Clicked install missing platform and follow guidance.

More prompts to install stuff, carry on.

10:59 Gradle then kicks off and downloads deps.

Gradle build completes.

Change the ip/bucket in the config as directed

- **this should really be abstracted to another static config file**

Click run project - I'm missing an emulator, so install one.

Then I have to download the image for the Pixel (I chose this) and Android 8.

Decide to try this on my galaxy s7, so I google how to get developer mode on and turn on USB debugging.

Aborted the virtual device installation and plugged in my phone, can now select it from menu in android studio.

Selected instant run, had to install android 7.0 compatability layer.

Check the sync-gateway output, notice it is complaining database doesn't exist.

- **FAILS on first run because sync gateway config is to call the database charlie - even if you change it to connect to demo bucket. In android project must leave /charlie as it is.**

So, set this back to correct ip and charlie.

Now it works. 11:10

Sync doesn't work from phone because L203 of the sync-gateway config is checking for the wrong document type from the phone.

- **Change shopping-basket to order**

Restart sync gateway with config. Now it all works. 11:18

