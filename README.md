# rag-23ai-demo

by Joe Hahn,<br />
joe.hahn@oracle.com,<br />
23 Oct 2024<br />
git branch=master

Demo about RAG on ADW/23ai

Note that all ocids, credentials, keys, passwords, compartments, etc noted below refer to my own, so you will need to modify those quantities to refer to your OCI tenancy details


### setup ADW

1 launch ADW

    region=chicago
    ecpus=4
    storage=1Tb
    compartment=orasenatdpltintegration03 (root)/BigData/SA/JoeHahn
    user=admin
    password=Welcome12345!
    url=https://cloud.oracle.com/db/adbs/ocid1.autonomousdatabase.oc1.us-chicago-1.anxxeljswe6j4fqa45qvwppf7twioilm45clltqo6xvcdw6looklxmg4almq?region=us-chicago-1

2 ADMIN user create user SAIRAG per https://snicholspa.github.io/tips_tricks_howtos/autonomous_database/select_ai_rag/?nav=close

    create user SAIRAG identified by "Welcome123456789!";
    grant dwrole to SAIRAG;
    grant unlimited tablespace to SAIRAG;
    grant execute on DBMS_CLOUD to SAIRAG;
    grant execute on DBMS_CLOUD_AI to SAIRAG;
    grant oml_developer to SAIRAG;


### clone repo to cloud shell

1 start cloud shell and confirm internet connectivity

    wget --spider www.oracle.com

2 create ssh key pair with no passphrase:

    ssh-keygen

3 add public ssh key to your Gitlab account at https://129.213.160.170/

    cat ~/.ssh/id_rsa.pub

with

    Title=rag-23ai-demo
    Fingerprint=98:a4:f5:b0:bb:ff:44:fd:dd:da:5e:5e:4a:e8:74:af

4 clone repo to cloud shell

    git clone git@129.213.160.170:jhahn/rag-23ai-demo.git
    cd rag-23ai-demo


### load data into ObjStore bucket

1 create bucket

    name=JoeHahnRagBucket
    region=chicago
    compartment=orasenatdpltintegration03 (root)/BigData/SA/JoeHahn

2 use cloud shell to create pem keys in ~/.oci, this key will allow ADW to talk to ObjStore

    oci setup config

3 display public key

    cat ~/.oci/oci_api_key_public.pem

and paste into OCI user's API keys

4 display private key

    cat ~/.oci/oci_api_key.pem

which is pasted into OML notebook and APEX app, below

5 delete /rag/text folder in ObjStore

6 use cloudshell to download 20 books (16Mb) from Project Gutenberg books, https://www.gutenberg.org/

    cd ~/rag-23ai-demo
    wget https://www.gutenberg.org/cache/epub/84/pg84.txt -O data/books/frankenstein.txt
    wget https://www.gutenberg.org/cache/epub/2701/pg2701.txt -O data/books/moby_dick.txt
    wget https://www.gutenberg.org/cache/epub/1342/pg1342.txt -O data/books/pride_prejudice.txt
    wget https://www.gutenberg.org/cache/epub/1513/pg1513.txt -O data/books/romeo_juliet.txt
    wget https://www.gutenberg.org/cache/epub/26184/pg26184.txt -O data/books/simple_sabotage.txt
    wget https://www.gutenberg.org/cache/epub/11/pg11.txt -O data/books/alice_wonderland.txt
    wget https://www.gutenberg.org/cache/epub/64317/pg64317.txt -O data/books/great_gatsby.txt
    wget https://www.gutenberg.org/cache/epub/45/pg45.txt -O data/books/green_gables.txt
    wget https://www.gutenberg.org/cache/epub/844/pg844.txt -O data/books/importance_earnest.txt
    wget https://www.gutenberg.org/cache/epub/1184/pg1184.txt -O data/books/monte_christo.txt
    wget https://www.gutenberg.org/cache/epub/2641/pg2641.txt -O data/books/room_view.txt
    wget https://www.gutenberg.org/cache/epub/174/pg174.txt -O data/books/dorian_gray.txt
    wget https://www.gutenberg.org/cache/epub/37106/pg37106.txt -O data/books/little_women.txt
    wget https://www.gutenberg.org/cache/epub/24022/pg24022.txt -O data/books/christmas_carol.txt
    wget https://www.gutenberg.org/cache/epub/43/pg43.txt -O data/books/jeckyll_hyde.txt
    wget https://www.gutenberg.org/cache/epub/1400/pg1400.txt -O data/books/great_expectations.txt
    wget https://www.gutenberg.org/cache/epub/345/pg345.txt -O data/books/dracula.txt
    wget https://www.gutenberg.org/cache/epub/2600/pg2600.txt -O data/books/war_peace.txt
    wget https://www.gutenberg.org/cache/epub/1661/pg1661.txt -O data/books/sherlock_holmes.txt
    wget https://www.gutenberg.org/cache/epub/768/pg768.txt -O data/books/wuthering_heights.txt

7 put 20 files into bucket

    ns=orasenatdpltintegration03
    bucket_name=JoeHahnRagBucket
    book=frankenstein.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=moby_dick.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=pride_prejudice.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=romeo_juliet.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=simple_sabotage.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=alice_wonderland.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=great_gatsby.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=green_gables.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=importance_earnest.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=monte_christo.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=room_view.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=dorian_gray.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=little_women.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=christmas_carol.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=jeckyll_hyde.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=great_expectations.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=dracula.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=war_peace.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=sherlock_holmes.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force
    book=wuthering_heights.txt ; bs_file=data/books/$book ; os_file=rag/doc/$book ; oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force


### Select-AI-RAG in OML notebook

1 use cloudshell to display private key

    cat ~/.oci/oci_api_key.pem

and paste into the RAG_CREDENTIAL portion of OML notebook RAG_books.dsnb

2 user SAIRAG execute OML notebook RAG_books.dsnb which will ask questions about the above books:

2A wuthering_heights questions:

    who did heathcliff fight?
    who was in love with heathcliff???

2B sherlock holmes questions

    what bird ate the blue carbuncle?
    what ate the blue stone?
    who did sherlock accuse of stealing the blue carbuncle?

2C moby dick

    what ship did ahab sail?
    what color is moby dick?

2D christmas_carol.txt 

    whats the matter with tiny tim?
    who was Scrooge's business partner?
    what did the ghost of christmas present look like?

2E great_gatsby.txt

    who does Jay Gatsby love?
    how did Myrtle die?

3 delete folder JoeHahnRagBucket/rag to delete all books from ObjStore

11 use cloud shell to upload War & Peace to Objstore

    ns=orasenatdpltintegration03
    bucket_name=JoeHahnRagBucket
    book=war_peace.txt
    bs_file=data/books/$book
    os_file=rag/doc/$book
    oci os object put --bucket-name $bucket_name --file $bs_file --name $os_file -ns $ns --force

12 execute CHECK_RAG.dsnb notebook to assess incompleteness in RAG answers


### APEX

1 log into OML notebooks as SAIRAG, import RAG_SETUP.dsnb, and execute

2 import f100.sql into APEX

3 open apex app

    url=https://gde727daa9b60fb-joehahn23ai.adb.us-chicago-1.oraclecloudapps.com/ords/r/sairag/vector-application-upload-download_1/login
    workspace=SAIRAG
    user=SAIRAG
    #password=Qwerty12345!
    password=Qwerty123456!

4 update APEX app so that it is aware of your bucket namespace, bucket name, user and tenancy ocids, the private key mentioned above, and ObjStore paths:

&nbsp;&nbsp;&nbsp;&nbsp;4.1 navigate from apex home > workspace utilities > web credentials > oci api access > update user ocid, private key, tenancy ocid, and fingerprint

&nbsp;&nbsp;&nbsp;&nbsp;4.2 navigate from OCI home > ObjStore bucket and create PAR

&nbsp;&nbsp;&nbsp;&nbsp;4.3 APEX home > open apex app > shared components > rest data sources > bucket v3 > remote server > split above PAR into url path (ends right after tenancy name) & url path prefix (rest of PAR)

&nbsp;&nbsp;&nbsp;&nbsp;4.4 open apex app > shared components > rest data sources > list buckets > parameters tab > edit compartment ocid where bucket lives

&nbsp;&nbsp;&nbsp;&nbsp;4.5 show all tab > check url path prefix

&nbsp;&nbsp;&nbsp;&nbsp;4.6 open apex app > shared components > rest data sources > list_objects_in_buckets > parameters tab > update bucket name

&nbsp;&nbsp;&nbsp;&nbsp;4.7 show all tab> confirm base url & url path prefix

&nbsp;&nbsp;&nbsp;&nbsp;4.8 open apex app > edit application definition > substitution tab > confirm BUCKET_PAR=entire PAR string

&nbsp;&nbsp;&nbsp;&nbsp;4.9 open apex app > edit page 12 > upper left corner 3rd icon > processing > create vector index > expand pl/sql code box on right > update location to point to bucket/path

5 use APEX to upload the manuals found in data/manuals

6 confirm files exist in /rag/text folder in Bucket

7 ask questions of each manual's troubleshooting sections:

&nbsp;&nbsp;&nbsp;&nbsp;7.1 chainsaw questions:

    chain saw doesn't start, what should i do?
    chain saw is making bad cuts, what should i do?

&nbsp;&nbsp;&nbsp;&nbsp;7.2 dryer questions

    my dryer does't work, what should i do?
    my clothes are shrinking, what should i do?
    dryer makes a thumping noise, what should i do?

&nbsp;&nbsp;&nbsp;&nbsp;7.3 seadoo questions

    sea-doo engine overheats, what should i do?
    sea-doo bilge has water in it, what should i do?

&nbsp;&nbsp;&nbsp;&nbsp;7.4 laptop questions

    computer won't turn on, what should i do?
    my laptop shows a flashing question mark, what should i do?

&nbsp;&nbsp;&nbsp;&nbsp;7.5 trimmer questions

    trimmer runs slow, what should i do?
    trimmer line feed isn't working, what should i do?


