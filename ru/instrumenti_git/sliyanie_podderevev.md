# Слияние поддеревьев

Теперь, когда вы увидели сложности системы подмодулей, давайте посмотрим на альтернативный путь решения той же проблемы. Когда Git выполняет слияние, он смотрит на то, что требуется слить воедино, и потом выбирает подходящую стратегию слияния. Если вы сливаете две ветви, Git использует _рекурсивную_ (recursive) стратегию. Если вы объединяете более двух ветвей, Git выбирает стратегию _осьминога_ (octopus). Эти стратегии выбираются за вас автоматически потому, что рекурсивная стратегия может обрабатывать сложные трёхсторонние ситуации слияния — например, более чем один общий предок — но она может сливать только две ветви. Слияние методом осьминога может справиться с множеством веток, но является более осторожным, чтобы предотвратить сложные конфликты, так что этот метод является стратегией по умолчанию при слиянии более двух веток.

Однако, существуют другие стратегии, которые вы также можете выбрать. Одна из них — слияние _поддеревьев_ (subtree), и вы можете использовать его для решения задачи с подпроектами. Сейчас вы увидите, как выполнить то же встраивание rack, как и в предыдущем разделе, но с использованием стратегии слияния поддеревьев.

Идея слияния поддеревьев состоит в том, что у вас есть два проекта, и один из проектов отображается в подкаталог другого и наоборот. Если вы зададите в качестве стратегии слияния метод subtree, то Git будет достаточно умён, чтобы понять, что один из проектов является поддеревом другого и выполнит слияние в соответствии с этим. И это довольно удивительно.

Сначала добавим приложение Rack в проект. Добавим проект Rack как внешнюю ссылку в свой проект, и затем поместим его в отдельную ветку:

	$ git remote add rack_remote git@github.com:schacon/rack.git
	$ git fetch rack_remote
	warning: no common commits
	remote: Counting objects: 3184, done.
	remote: Compressing objects: 100% (1465/1465), done.
	remote: Total 3184 (delta 1952), reused 2770 (delta 1675)
	Receiving objects: 100% (3184/3184), 677.42 KiB | 4 KiB/s, done.
	Resolving deltas: 100% (1952/1952), done.
	From git@github.com:schacon/rack
	 * [new branch]      build      -> rack_remote/build
	 * [new branch]      master     -> rack_remote/master
	 * [new branch]      rack-0.4   -> rack_remote/rack-0.4
	 * [new branch]      rack-0.9   -> rack_remote/rack-0.9
	$ git checkout -b rack_branch rack_remote/master
	Branch rack_branch set up to track remote branch refs/remotes/rack_remote/master.
	Switched to a new branch "rack_branch"

Теперь у нас есть корень проекта Rack в ветке `rack_branch` и наш проект в ветке `master`. Если вы переключитесь на одну ветку, а затем на другую, то увидите, что содержимое их корневых каталогов различно:

	$ ls
	AUTHORS	       KNOWN-ISSUES   Rakefile      contrib	       lib
	COPYING	       README         bin           example	       test
	$ git checkout master
	Switched to branch "master"
	$ ls
	README

Допустим, вы хотите поместить проект Rack в подкаталог своего проекта в ветке `master`. Вы можете сделать это в Git'е командой `git read-tree`. Вы узнаете больше про команду `read-tree` и её друзей в главе 9, а пока достаточно знать, что она считывает корень дерева одной ветки в индекс и рабочий каталог. Вам достаточно переключиться обратно на ветку `master` и вытянуть ветку `rack` в подкаталог `rack` основного проекта из ветки `master`:

	$ git read-tree --prefix=rack/ -u rack_branch

После того как вы сделаете коммит, все файлы проекта Rack будут находиться в этом подкаталоге — будто вы скопировали их туда из архива. Интересно то, что вы можете довольно легко слить изменения из одной ветки в другую. Так что если проект Rack изменится, вы сможете вытянуть изменения из основного проекта, переключившись в его ветку и выполнив `git pull`:

	$ git checkout rack_branch
	$ git pull

Затем вы можете слить эти изменения обратно в свою главную ветку. Можно использовать `git merge -s subtree` — это сработает правильно, но тогда Git, кроме того, объединит вместе истории, чего вы, вероятно, не хотите. Чтобы получить изменения и заполнить сообщение коммита, используйте опции `--squash` и `--no-commit` вместе с опцией стратегии `-s subtree`:

	$ git checkout master
	$ git merge --squash -s subtree --no-commit rack_branch
	Squash commit -- not updating HEAD
	Automatic merge went well; stopped before committing as requested

Все изменения из проекта Rack слиты и готовы для локальной фиксации. Вы также можете сделать наоборот — внести изменения в подкаталог `rack` вашей ветки `master`, и затем слить их в ветку `rack_branch`, чтобы позже представить их мейнтейнерам или отправить их в основной репозиторий проекта с помощью `git push`.

Для получения разности между тем, что у вас есть в подкаталоге `rack` и кодом в вашей ветке `rack_branch`, чтобы увидеть нужно ли вам объединять их, вы не можете использовать нормальную команду `diff`. Вместо этого вы должны выполнить `git diff-tree` с веткой, с которой вы хотите сравнить:

	$ git diff-tree -p rack_branch

Или, для сравнения того, что в вашем подкаталоге `rack` с тем, что было в ветке `master` на сервере во время последнего обновления, можно выполнить:

	$ git diff-tree -p rack_remote/master