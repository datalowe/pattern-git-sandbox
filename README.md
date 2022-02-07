(there's an English version of this guide available in this repo's file 'README_translated.md', and [at datalowe.com](https://datalowe.com/post/git-collaboration/))

# Git sandbox 
Här testar vi saker med git. Nedanför finns alla tips från Lowe på Discord samlade.

## Grenar, generellt
> Har du möjlighet att skriva en superkort github-guide åt oss som vi kan pinna här? Jag kan inte t ex branching alls och det känns viktigt att lära sig nu när vi är så många inne i repona

De viktigaste kommandona är

`git branch foo` skapa ny gren lokalt
`git checkout foo` byt över till specificerad gren (alltså skifta från ex master till foo)

När man väl är på en gren så kommer allt man gör vara kopplat enbart till den grenen. Om jag gör `git add .` och `git commit -m "msg"` så kommer enbart grenen jag är på att påverkas. Om jag sen byter över `git checkout master` så kommer jag att märka att alla filer 'rullas tillbaka' till det state/innehåll de var i innan jag bytte till foo.

Det är bra att känna till att alla olika 'states'/versioner av filer egentligen är lagrade i mappen '.git', det är ingen magi, och det är därför det är viktigt att aldrig radera '.git'-mappen.

När jag jobbat på länge i foo och är nöjd med resultaten, så att jag känner att jag vill föra in dem i master, så kan jag göra på två olika sätt.

**Första sättet**
Göra `git push origin foo`. Det här skapar en ny gren på 'origin' (dvs GitHub i vårt fall), och skickar lokala 'foo':s innehåll/state dit. master-grenen har fortfarande inte påverkats nu. 

Om jag nu går till github.com, till repository:n, så kan jag se att det finns två grenar där: 'master' och 'foo'. Om jag går in i 'foo' så ser jag att den 'ligger före' med x antal commits jämfört med master. Om jag klickar på 'Contribute' (ganska nära toppen) så kan välja att göra en 'Pull Request' (PR). 

I och med min PR 'ber jag om lov' att få föra in commits från foo till master. Det ger en bra möjlighet att överblicka vad man gjort för ändringar innan man genomfört dem.

När PR:en väl godkänts och förändringar förts in i master, kan man lokalt köra `git checkout master` och `git pull origin master` för att ändringarna ska komma till ens lokala kopia/repo.

Jag skulle rekommendera att vi i regel håller oss till första sättet.

**Andra sättet**
Köra `git checkout master` och sedan `git merge --ff-only foo`. Då förs ändringar från foo direkt in i master lokalt. Sen kan man pusha till github med `git push origin master`.


## Förebygga problem med 'divergent branches', att grenar kommer i osynk på oväntade sätt
> jag gjorde en git pull - öppnade mappen i vs och en fil var gul (den xxx gjort en förändring i).
> Arbetade i repot. Körde git add . och sedan git push men där tog det stopp.
> Något tips?

Jag vet inte just vad som blev fel i det här fallet, men här är tips om vad man kan försöka göra för att undvika/debugga problem:

`git branch` (alltså utan några argument) För att kontrollera vilken branch man är på och vilka andra man har lokalt. Den aktiva grenen är markerad med en stjärna (och ofta i grönt, beror på CLI-program tror jag). 

`git status` För att kontrollera vilken branch man är på, ifall man har några filer som inte stage:ats (inte lagts till med `git add`) än, vad man har stage:at (kört `git add` men inte `git commit` med, och alltså behöver commit:a innan de kommer följa med vid `git push`), och hur ens egna grens status ligger till jämfört med den kopplade 'remote'(Github)-grenen. Så här t ex
```bash
> git status
On branch main
Your branch is up to date with 'origin/main'.
```
Det här betyder att jag är på grenen main lokalt, och den är kopplad till 'remote'(Github)-grenen `main`. Jag har inga lokala commits som inte skickats till remote än, och **vad mitt lokala Git vet** så har inte remote-grenen några commits som min lokala repo saknar. Om det stått typ `your branch is behind origin/main by 3 commits and can be fast-forwarded` så hade jag helt enkelt kunnat köra (efter dubbelkoll med `git fetch`, se nedan) `git pull origin main --ff-only` för att 'komma ikapp' GitHub-grenen i mit lokala repo:s gren. Om det stått `is behind...` och *inte* slutat med `can be fast-forwarded` så finns antagligen en konflikt mellan commits på den lokala grenen och commits på GitHub. Då blir det krångligare, fråga gärna mig om jag är online.

`git fetch origin foo` Använd det här för att hämta hem uppdaterad information om vilka commits som remote-grenen (`origin/foo`, alltså grenen `foo` på GitHub) har. Kör nu `git status` igen - det kan visa sig att det finns commits på GitHub som lokala Git inte kände till . **I regel så är det en bra vana att köra `git fetch` innan `git status` för att vara säker på att man inte missar några commits på GitHub**

Innan `git pull` eller `git push`, kör `git fetch` och `git status`. Och istället för 'bara' `git pull` t ex, kör `git pull origin <branch>`, där `<branch>` är namnet på den 'GitHub-gren' som du vill hämta data ifrån. Med `git status` ser du dels att du är på rätt gren lokalt, dels att det inte finns några 'saknade' commits lokalt (och/eller konflikter). Med `git pull/push origin <branch>` säkerställer du att du interagerar med rätt gren på GitHub (annars antar Git att du vill göra saker med den lokala grenens kopplade 'remote'-gren, dvs den som det står om i `git status`-output, vilket kanske inte är det du vill).

## Exempel på flöde med att jobba på gren lokalt och pusha till sen PR:a på GitHub
> nu ligger ju bara master/origin på github för frontend.
> 
> Det jag gör då är att jag kör en git pull origin <branch> main(?),, så jag har senaste. Sedan skapar jag en git branch tables (exempel) där jag arbetat med mina tables. kan kommita flera gånger (och skapa taggar?) och sedan när man är nöjd med koden så mergar man
>
> Rent krasst kan ni alltså dra ned den branchen - fortsätta arbeta med den och pusha commits. och när vi är nöjda så mergar man.

Ja, så i detalj:

I början har du
* På GitHub grenen master (eller 'main' - var noggrann med att kolla, för 'main' är standardnamnet på nya GitHub-projekt sen förra året, och även i senaste versionerna av Git vad jag förstår)
* I din lokala repo grenen master (om du gjort git clone t ex)

När du lokalt kör `git pull origin master` så säkerställer du att din lokala 'master'-gren har de allra senaste commits från GitHub-grenen.

Du kör `git branch tables`. Den här nya grenen delar nu commithistorik med din lokala 'master' (och därmed även 'master' på GitHub förstås, eftersom du syncat dem med pull).

Du kör `git checkout tables` för att växla över till din lokala 'tables'-gren. (kan kontrolleras med `git branch`)

Du ändrar en fil 'foo.txt'. Om du kör `git status` här så ser du att du 1) är på lokala grenen 'tables' 2) att 'tables' inte är kopplad till någon remote (GitHub) gren än, och 3) det finns en ändring i 'foo.txt' som inte är i staging area.

Du kör `git add foo.txt`. Om du nu kör `git status` märker du att 'foo.txt' lagts till i staging area, men inte commit:ats än.

Du kör `git commit -m 'add foo.txt'`. Om du nu kör `git status` får du veta att `nothing to commit, working tree clean`, eftersom det inte finns några 'ej stagade' eller 'stagade men ej committ:ade' ändringar.

Du kör `git push origin tables`. Det här skapar en gren 'tables' på GitHub som delar commithistorik med lokala 'tables'-grenen.

Via GitHubs webbgränssnitt så går du till ditt repo, växlar över till 'tables'-grenen, och väljer (nära toppen) att du vill starta en pull request (PR).

Du godkänner PR på GitHub. 'master' på GitHub har nu 'spolats framåt' i commithistorik så att den kommit ikapp 'tables'

Tillbaka i ditt CLI kör du `git checkout master`. Den här lokala grenen vet inget om vad som hänt i 'tables' eller på GitHub (syns m `git status`/`git log`). Om du däremot kör `git fetch origin master` och sen `git status` ses ändringarna. Kör `git pull origin master` för att 'komma ikapp'.

Jag gjorde även en video som går igenom flödet ovan ('git-sandbox-example.mkv' i det här repot), med lite extra detaljer. Den är utan ljud, jag försökte skriva kommentarer direkt i CLI:t där det verkade passa. Ursäkta kvalitén, jag missade öka textstorleken så kan vara svårt att läsa ibland.

## Döpa grenar och systematik i commits
> i did a branch häromdagen(!!!) och döpte den till table. Men rent krasst blev den inte riktigt så clean som jag hade tanke på - blev en hel del andra ändringar också. Något konkret tips på hur man döper branches och eller ska tänka när man parallellt behöver korrigera fler saker samtidigt? 

Jag tycker också det är svårt att döpa grenar, och jag tror det är extremt vanligt att man fixar diverse småsaker man stöter på medan man jobbar på en gren, det är bara bra att man löser saker när man märker dem istället för att glömma bort. Men gren-namnet är inte alls särskilt viktigt. När du väl gjort en PR och merg:at med main/master så kommer du ta bort grenen, och gren-namnet kommer bara att synas i commitmeddelandet kopplat till PR:en. Med andra ord kommer man nästan aldrig se det efteråt. Jag använder grennamnet mest som en påminnelse till mig själv om vad det är jag jobbar på. Om jag skapar en gren 'feature/login' t ex så kan jag ifall jag tappar tråden eller går in på sidospår för mycket skriva `git branch` och påminna mig själv 'just det, det är implementation av login som är huvudmålet'. Sen är det väl en fråga hur stort arbetsområde man ska ta sig an, ifall man vill göra det i etapper ('feature/login_view' eller 'feature/login_route' t ex). Men det handlar mer generellt om hur man planerar jobbet.

Typ 'feature/login' el 'debug/login' kan hjälpa en hålla reda på om man jobbar på nåt nytt, eller med att fixa något redan existerande.

Det som är desto viktigare är att försöka göra commits ofta och med tydliga commit-meddelanden. Jag skriver alltid i imperativ, det är standarden för git vad jag förstått (konstigt att det inte kommit upp i nån kurs) och hjälper mig att hålla det konsekvent. Så till exempel `git commit -m 'add scooter PUT route and corresponding controller'`, istället för `... 'added scooter PUT...'`. Men tempus är inte så noga, det viktiga är att försöka commit:a ofta och gärna någorlunda sammanhållet (istället för `... -m 'fix CSS for login, registration, customer overview views, fix stuff in routing, add Customer service with a bunch of functions...'`). Det hjälper ofta att använda `git status`,  bara lägga till vissa filer med `git add`, commit:a, lägga till, commit:a, osv (istället för `git add .`).

