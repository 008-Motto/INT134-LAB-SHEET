# INT134 System Deployment
## Class 06 Introduction to Container

### Lab Instructions

1. It is best to learn from experiment, read command help, and figure out the solution yourself.

2. Do not search for the answer on internet or use AI.

3. Do not ask or copy answers from your friend.

---

### Setup

1. Connect to your `int134` VM with ssh.

2. Create a directory `~/int134/lab06class/` under your home directory and change working
    directory to this directory.

3. Copy `/home/public/labs/lab06class/answers.yaml` to `~/int134/lab06class/answers.yaml`.

4. Modify `answers.yaml`: set your student ID and your full name.

**You need class 5 working.** `https://<your VM>/school/` must show the register in a browser,
and `https://<your VM>/school/api/students` must return JSON. Nothing is installed today — the
engine you need is already on this machine, and you have never been allowed to speak to it.

**One number is yours all afternoon.** Class 4 gave your API `3` followed by the last three
digits of your VM's name, and class 5 gave your page `8` and the same three digits. Today a
container gets `9` and the same three digits. On `lvm68136` that is `9136`; substitute your own
everywhere you see `<port>`.

**`sudo` will ask for your password, several times today.**

---

### What this VM records about your work

While you work on this lab, your VM notes **how many minutes you were active** — nothing
more. A minute counts as active if **your terminal produced output** in that minute.

**What is recorded:** a count of active minutes, and nothing else.

**When:** during the class session, and afterwards until you have finished the lab or one
week has passed, whichever comes first.

**What is *not* recorded:** your commands, your keystrokes, your files, your IP address,
and anything at all during a minute when nobody is connected.

**Editing over VS Code Remote-SSH does not count.** VS Code is not recommended for this
course: its autocomplete frequently supplies the answer, which is not what you are here to
practise. It also opens no terminal of its own, so time spent editing files through it
earns no participation credit. Work at a terminal — including a terminal opened inside VS
Code, which counts normally.

**What it is used for:** your participation credit — which rewards working in class,
whether or not you finish — and measuring how long this lab really takes, so that future
labs are better sized.

You can read exactly what your own VM has recorded at any time:

```bash
cat /var/lib/int134/activity.log
cat /var/lib/int134/activity.counter
```

**These two files are the record of your participation. Altering them scores zero
participation for this lab.** Both are checked against each other, so an edit to either one
shows up. If you think the record is wrong — and it can be, this is software — **tell me**.
A mistake corrected is nothing; an edited file is academic dishonesty.

---

### What today is

Everything you have deployed so far, you installed. Today you run software you did not install,
in a box you did not build, and you will find out the hard way what is inside that box and what
is not.

Your school page moves into a container. The page in the browser does not change at all — that
is the point of the part where nginx stops reading files and starts forwarding instead. Then you
throw the container away, and find out what went with it.

Seven things, in order. **Do them in order**; each one is only interesting because of the one
before it.

| | You will | And it leaves |
| :--- | :--- | :--- |
| 1 | Get permission to speak to the engine | a command that works, after a second try |
| 2 | Find out where images come from, when the internet says no | a machine that can fetch images |
| 3 | Run a container, and look at it from four directions | a web server you did not configure |
| 4 | Put your page inside it | your page, on a port of its own |
| 5 | Make nginx forward `/school/` to it | a browser that notices nothing |
| 6 | Destroy it, and start another one the same way | a question you can only answer now |
| 7 | Put your page back | a machine that works, and a lesson |

**Run `int134 check lab06class` at the end of every part.** It records what you have done so
far. Nothing you do later takes away what an earlier part earned.

---

### Part 1 - The engine, and who may speak to it

There is a program running on this machine that you have never used. It has been there since
before class 1. It is not a service you start and stop today — it is already running, and the
only thing standing between you and it is permission.

| Command | What it does |
| :--- | :--- |
| `docker ps` | List the containers that are running |
| `docker version` | Say what the client and the engine are |
| `id` | uid, gid and groups of the user running this process |

1. Run `docker ps`.

    ```
    permission denied while trying to connect to the docker API at unix:///var/run/docker.sock
    ```

    *Read that carefully. It is not "command not found" and it is not "no such file". The socket
    is there, the engine is running, and you are not allowed to open it.*

2. Find out which group owns `/var/run/docker.sock`, then add your own account to that group.
    You did this in class 1, to a different account and a different group.

3. Run `docker ps` again.

    *It is still refused. Nothing went wrong in step 2 — check with `id` and you will not see
    the new group there either. A group you have just been given is not in a session that was
    already open before you were given it.*

4. Get the group into this session — either by starting a new login shell that has it, or by
    reconnecting — and run `docker ps` once more.

    ```
    CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS   PORTS   NAMES
    ```

    *An empty list is the right answer. There are no containers on this machine yet.*

---

### Part 2 - Where images come from

A container is made from an **image**, and an image is fetched from a **registry**. The default
registry is Docker Hub, on the internet, and it does not give them away without limit.

Sixty-eight of you share one address as far as Docker Hub is concerned.

| Command | What it does |
| :--- | :--- |
| `docker pull <image>:<tag>` | Fetch an image into this machine's own store |
| `docker images` | List the images this machine has fetched |
| `docker run <image>` | Make a container from an image and start it |

5. Pull `nginx:1.30-alpine`.

    *Some of you will get it. Some of you will not, and which is which is nothing to do with
    you — keep whatever it printed on your screen.*

6. Now pull `hello-world`, all of you, at the same time.

    ```
    Using default tag: latest
    latest: Pulling from library/hello-world
    <...>
    ```

    **What one word did the engine use to refuse it?** If your pull was not refused, write
    `not refused` instead. **→ `T6`**

    *Answer shape: one word.*

7. There is a second registry inside the university, and it keeps a copy of everything it has
    ever been asked for. Your machine does not know about it yet.

    Put the file `/home/public/labs/lab06class/daemon.json` at `/etc/docker/daemon.json`.
    **Copy it — do not retype it.** It is JSON, and an engine given broken JSON does not start.

    That file names the registry. It also caps how much disk one container's output may
    take, which matters in a later class when you meet a container that restarts for ever.

    Then make the engine read it. Changing a configuration file has never been enough on this
    machine, in any class.

    ***If the engine will not start, read its status before changing anything else.*** *It will
    not start at all on JSON it cannot parse — that is why you copied the file. And once it has
    failed a few times in a row, systemd stops trying and answers `Start request repeated too
    quickly` however good your file now is: you have to clear the failed state before starting it
    again. Measured on a real machine — a correct file and a refusal to start, at the same time.*

8. Ask the engine to describe itself, and find the two lines that matter: the registry it has
    been told to use, and the storage driver it keeps images with.

    ```
     Server Version: 29.1.3
     Storage Driver: <...>
     Registry Mirrors:
      https://lvmolarn.sit.kmutt.ac.th/
    ```

    **What storage driver does the engine on your machine use?** **→ `T8`**

    *Answer shape: one word. Your `Server Version` may differ from the sample.*

9. Pull `nginx:1.30-alpine` and `hello-world` again. Both arrive.

    Then run `hello-world`. It is the smallest container in the world and it exists only to
    print a paragraph and stop.

    *Nothing is asking Docker Hub any more. The copy inside the university answers instead, and
    it has no limit on you.*

---

### Part 3 - A container of your own

`nginx:1.30-alpine` is a web server somebody else installed and configured. You are about to
run it without configuring anything at all.

| Command | What it does |
| :--- | :--- |
| `docker run -d` | Start it and leave it running in the background |
| `docker run --name <name>` | Call it something, so you can refer to it |
| `docker run -p <address>:<host port>:<container port>` | Let this machine's network reach a port inside it |
| `docker logs <name>` | Show what the container has printed |
| `docker exec <name> <command>` | Run a command inside a container that is already running |
| `docker stop` / `docker start` | Stop it; start it again |

10. Start a container from `nginx:1.30-alpine`, in the background, named `school-web`, with its
    port 80 reachable at `127.0.0.2:<port>` on this machine.

    ```
    27f07e98a34aad197c944107449f14c0d738b5dc8a6fa642af76a6cf11964d6d
    ```

    *That long string is the container's real name. `school-web` is a label for your
    convenience.*

    ***`127.0.0.2` is not a typo.*** *The whole of `127.0.0.0/8` is this machine talking to
    itself, not just `127.0.0.1`. You are giving this container an address of its own.*

11. Ask for `http://127.0.0.2:<port>/` with `curl`.

    ```
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    ```

    *A working web server, in one command, that you did not install and cannot see in
    `systemctl`. It is not in `/etc/nginx` either — go and look.*

12. List the running containers.

    ```
    CONTAINER ID   IMAGE               STATUS         PORTS                    NAMES
    27f07e98a34a   nginx:1.30-alpine   Up 2 minutes   127.0.0.2:9136->80/tcp   school-web
    ```

13. Show what the container has printed since it started.

    ```
    2026/09/14 10:49:39 [notice] 1#1: start worker process 21
    172.17.0.1 - - [14/Sep/2026:10:49:39 +0000] "GET / HTTP/1.1" 200 896 "-" "curl/8.5.0" "-"
    ```

    *That is the request you made in step 11, in an access log, on a machine with no
    `/var/log/nginx`. And look at the address it thinks you came from — it is not yours. Class
    5 taught you what that problem is called; class 10 fixes it here.*

14. Run a command **inside** the container: list its document root, `/usr/share/nginx/html`.

    ```
    -rw-r--r--    1 root     root           497 Jul 15 18:38 50x.html
    -rw-r--r--    1 root     root           896 Jul 15 18:38 index.html
    ```

    *Two files you have never seen, owned by a root that is not quite this machine's root. This
    is the page that answered step 11.*

15. Stop the container, ask for the page again, then start it and ask once more.

    *A stopped container is still on this machine. It has not been deleted, and `docker ps` is
    not the command that shows it.*

---

### Part 4 - Your page, inside it

16. Copy the **contents** of `/var/www/school/` into the container's document root.
    `docker cp` takes a source and a destination, and one of the two is
    `school-web:/usr/share/nginx/html/`.

    *It prints nothing at all when it works, whether or not it did what you wanted. Check by
    looking inside, as in step 14 — that is the only thing that will tell you.*

    ***The trailing-slash rule from class 3 does NOT hold here, and this is the one place in the
    course where a habit works against you.*** *With `rsync`, `src/` meant the contents of `src`.
    `docker cp` does not read it that way: to `docker cp`, both `src` and `src/` mean the directory
    itself, and only `src/.` means its contents. Copy the directory by mistake and you get a
    directory inside the document root, nginx keeps serving the page the image ships with, and
    nothing reports an error.*

    *If step 14's listing shows a directory named `school`, that is what happened.*

17. Ask for `http://127.0.0.2:<port>/` again.

    ```
    <!doctype html>
    <html lang="en">
    <head>
      <meta charset="utf-8">
    ```

    *Your page, served by a web server you did not install, on an address of its own.*

---

### Part 5 - The door

Your page is now in two places: `/var/www/school/` on this machine, where nginx reads it, and
inside the container, where nobody is asking for it. `https://<your VM>/school/` still comes
from the first one.

| Directive | What it does |
| :--- | :--- |
| `proxy_pass http://127.0.0.2:<port>/;` | Send this request on to something else and return its answer |
| `alias /var/www/school/;` | Read files from this directory and return them |

18. In `/etc/nginx/conf.d/school.location`, make `/school/` **forward** to the container
    instead of reading files from disk. The directive you need is the one you wrote in class 5;
    only its destination is new, and the trailing slash on it matters for the same reason it
    did then.

    Test the configuration and reload nginx. Do not skip the test.

    ***Leave `/school/api/` exactly as it is.*** *It forwards to your API and has done since
    class 5. Today's work is about the page.*

19. Open `https://<your VM>/school/` in a browser.

    *Nothing has changed. The page looks the same, the register still loads, and the address bar
    is identical. That is not a disappointment — it is the entire reason reverse proxies exist,
    and you have just moved a site onto completely different software without a single visitor
    noticing.*

    Now read the container's log again. Your browser's request is in it.

    *The page did not change. Where it came from did. Only the container can tell you that, and
    only because you asked the thing that received the request rather than the thing that
    answered you.*

20. Check that `https://<your VM>/school/api/students` still returns JSON.

---

### Part 6 - Take it away

21. Delete the container — not stop it, delete it — and ask for `https://<your VM>/school/`
    again.

    ```
      /school/ now answers 502
    ```

    *`502` is nginx saying "I forwarded it and nothing answered". Your page is untouched in
    `/var/www/school/`, and it is not what is being served any more.*

22. Start a fresh container from the same image, the same way as step 10, and ask for
    `https://<your VM>/school/` once more.

    **What did `/school/` show this time?** **→ `T22`**

    *Answer shape: one line.*

    *You used the same image, the same name, the same port and the same command. Read the new
    container's log too: it is empty apart from its own start-up. This is a different container
    that happens to have the name you reused.*

23. Put your page back, the same way you did in step 16, and check `/school/` one last time.

    *Everything is working again, and you have repaired it by hand. Next week you stop doing
    that: an image you build yourself has your page in it already, and a container made from it
    is right the moment it starts.*

---

### Finish

Run `int134 check lab06class` one more time before you leave.

Leave the container running and `/school/` forwarding to it. Next week starts here.

---

### What is checked

**On your machine.**

| Part | What is checked |
| :--- | :--- |
| 1 | your account is in the `docker` group; the session you are pushing from has it too |
| 2 | `/etc/docker/daemon.json` names the internal registry **and** the engine has read it; the nginx image is on the machine |
| 3 | a container built from `nginx:1.30-alpine` is running |
| 4 | your page is inside that container, **at the path nginx serves from** — not merely somewhere beneath it |
| 5 | `/school/` is a proxy rather than a document root, **and** a request to it really arrives inside the container |
| 5 | the register still loads: `/school/api/students` answers, **and** the copy of the page now inside the container still knows where to ask for it |

**From the network**, on my collector, against your VM — from outside it, which is the only
place some of this can be seen from: that `https://<your VM>/school/` returns your page, and
that your container's **own port is not open to the world**. A container published to one
address on this machine is unreachable from the network; published to all of them it is not,
and nothing on the VM itself can tell you which you did.

*You will see my request in your container's log. It is marked, and it did not come from you.*

**From your answer file.** `T6`, `T8` and `T22`.

**No marks are shown, and no score is ever displayed to you.**

**Part 6 is not checked at all, deliberately.** Everything it teaches is gone by the time you
finish step 23 — which is the whole point of it — so a check on it would have to be false for
every student who finished the class. `T22` is how it is collected instead.

**The register is checked by asking the container, not nginx.** nginx has served those files from disk since class 5, so a check that read them through `/school/` would pass on a machine where nothing had been done. Step 16 says *contents* for a reason: the page is more than `index.html`, and one of its files is not in the repository.

**Nothing checks which name you gave your container.** Call it anything. The checks look for the
image it was built from, because next week Compose starts naming containers for you.

---

### Without the sheet

- A web server can run on a machine that has no web server installed.
- An image is fetched once; a container is made from it and can be thrown away.
- What you put into a container by hand is in the container, and not in the image.
- A site can be moved onto entirely different software without its visitors noticing.
- The only witness to where a request really went is the thing that received it.
