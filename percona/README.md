# Data Masking With Percona

1. Start container
    ### Run docker
    ```
    docker run -d --name ps -e MYSQL_DATABASE=percona_db -e MYSQL_ROOT_PASSWORD=root percona/percona-server:8.0
    docker exec -it ps /bin/bash
    ```
---
2. Experiment : Basic usage/SSN numbers/Credit card numbers และการใช้ View
    ### Installation
    ติดตั้ง Plugin เพื่อสำหรับใช้งาน Function data masking ใน Database
    ```
    INSTALL PLUGIN data_masking SONAME 'data_masking.so';
    ```
    
    ### Basic usage
    เป็นการใช้งาน Data masking แบบพื้นฐาน โดยจะมีด้วยกัน 2 รูปแบบ คือ MASK_INNER(รูปแบบปกปิดระหว่างข้อมูล) และ MASK_OUTER(รูปแบบปกปิดข้างหน้า-ข้างหลังข้อมูล)
    ```
    use percona_db;
    create table sensative_data (id int, hushhush bigint);
    insert into sensative_data values (1,1234567890),(2,0987654321);
    select id, 
        hushhush as 'Original', 
        MASK_INNER(convert(hushhush using binary),2,3) as 'Inner', 
        MASK_OUTER(convert(hushhush using binary),3,3) as 'Outer' 
    from sensative_data;
    ```

    ### SSN numbers (Social Security Numbers)
    เป็น Function ที่ใช้เฉพาะสำหรับปกปิดเลขประกันสังคม 
    ```
    create table employee (id int, name char(15), ssn char(11));
    insert into employee values (1,"Moe","123-12-1234"), (2,"Larry","22-222-2222"),(3,'Curly',"99-999-9999");
    select id, 
        name, 
        mask_outer(name,1,1,'#') as 'masked', 
        mask_ssn(ssn) as 'Masked SSN' 
    from employee;
    ```

    ### Credit card numbers
    เป็น Function ที่ใช้เฉพาะสำหรับปกปิดเลขบัตรเครดิต
    ```
    alter table employee add column cc char(16);
    update employee set cc = "1234123412341234";
    select mask_pan(cc) from employee;
    ```
---
3. ลองใช้ Robot framework เช่น insert, select
---
## Link reference
1. https://docs.percona.com/percona-server/8.0/installation/docker.html
2. https://www.percona.com/blog/data-masking-with-percona-server-for-mysql-an-enterprise-feature-at-a-community-price/