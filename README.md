# rag-23ai-demo-archive

by Joe Hahn,<br />
joe.hahn@oracle.com,<br />
kevin.ortiz@oracle.com,<br />
1 July 2025<br />
git branch=master

Demo of Select-AI-RAG on ADW/23ai

This demo-archive is associated with Oracle blog post titled
"In-Database Retrieval Augmented Generation (RAG) using Select-AI-RAG in Oracle’s Autonomous Data Warehouse "
published at {url-tbd} 

Note that all ocids, credentials, keys, passwords, compartments, etc noted below are referring to those used by the authors of this content, 
and that you will need to modify those quantities if you wish rebuild this demo in your OCI tenancy via the following steps:

1 launch ADW in your OCI tenancy

2 ADW user ADMIN create user SAIRAG with necessary permissions

    create user SAIRAG identified by "<password>";
    grant dwrole to SAIRAG;
    grant unlimited tablespace to SAIRAG;
    grant execute on DBMS_CLOUD to SAIRAG;
    grant execute on DBMS_CLOUD_AI to SAIRAG;
    grant oml_developer to SAIRAG;

per https://snicholspa.github.io/tips_tricks_howtos/autonomous_database/select_ai_rag/?nav=close

3 start an OCI cloud shell session, and clone this github repo to cloud shell

    git clone https://github.com/oracle-nace-dsai/rag-23ai-demo-archive.git

4 use cloud shell to create a pem-style ssh key-pair in ~/.oci, this key will allow ADW to talk to ObjStore and the GenAI service

    oci setup config

5 display public key

    cat ~/.oci/oci_api_key_public.pem

and paste into your OCI user's API keys

6 display private key

    cat ~/.oci/oci_api_key.pem

which will be pasted into the OML notebook and APEX app that are described below

7 create an Object Store bucket named RagBucket

8 use cloudshell to download 20 books (16Mb) from Project Gutenberg books, https://www.gutenberg.org/

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

9 put 20 files into bucket

    ns=orasenatdpltintegration03
    bucket_name=RagBucket
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

10 user SAIRAG import OML notebook RAG_books_archive.dsnb

11 use cloudshell to display private key

    cat ~/.oci/oci_api_key.pem

and paste that into the RAG_CREDENTIAL paragraph of OML notebook RAG_books_archive.dsnb

12 revise code paragraph in RAG_books_archive.dsnb that creates RAG_CREDENTIAL, namely update the OCIDS and the key fingerprint referenced there

13 the next paragraph uses DBMS_CLOUD.LIST_OBJECT() to confirm that the notebook can list the 20 books stored in Object Store, so update the Object Store path used there

14 the next paragraph creates profile RAG_PROFILE, and that requires updating oci_compartment_id

15 the next paragraph creates RAG_INDEX which requires an updated Object Store path

17 execute OML notebook RAG_books_archive.dsnb to build and and then test the Select-AI-RAG pipeline in ADW/23ai

18 import APEX application f100.sql into APEX

19 user SAIRAG opens apex app...does user need to create a workspace? use the SAIRAG workspace?

20 revise the APEX app so that it is aware of your bucket namespace, bucket name, user and tenancy ocids, the private key mentioned above, and the ObjStore paths, via the following steps:

&nbsp;&nbsp;&nbsp;&nbsp;A. navigate from apex home > workspace utilities > web credentials > oci api access > update user ocid, private key, tenancy ocid, and fingerprint

&nbsp;&nbsp;&nbsp;&nbsp;B. navigate from OCI home > ObjStore bucket and create PAR

&nbsp;&nbsp;&nbsp;&nbsp;C. APEX home > open apex app > shared components > rest data sources > bucket v3 > remote server > split above PAR into url path (ends right after tenancy name) & url path prefix (rest of PAR)

&nbsp;&nbsp;&nbsp;&nbsp;D. open apex app > shared components > rest data sources > list buckets > parameters tab > edit compartment ocid where bucket lives

&nbsp;&nbsp;&nbsp;&nbsp;E. show all tab > check url path prefix

&nbsp;&nbsp;&nbsp;&nbsp;F. open apex app > shared components > rest data sources > list_objects_in_buckets > parameters tab > update bucket name

&nbsp;&nbsp;&nbsp;&nbsp;G. show all tab> confirm base url & url path prefix

&nbsp;&nbsp;&nbsp;&nbsp;H. open apex app > edit application definition > substitution tab > confirm BUCKET_PAR=entire PAR string

&nbsp;&nbsp;&nbsp;&nbsp;I. open apex app > edit page 12 > upper left corner 3rd icon > processing > create vector index > expand pl/sql code box on right > update location to point to bucket/path

21 use the APEX app to ask additional questions of the Select-AI-RAG knowledge base

22 email any questions to joe.hahn@oracle.com and kevin.ortiz@oracle.com 
