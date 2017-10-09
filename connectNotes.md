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


