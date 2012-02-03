---
layout: post
title: "Mysql Enum and Test::*"
date: 2012-02-03 19:06
author:yongbin
comments: true
categories: [mysql, test]
---

MySQL enum과 perl Test:: 이야기
====================


MySQL의 enum column
-------------------

 제가 MySQL을 사용한지 약 10년전 정도의 시간이 지났습니다. 처음 MySQL을
사용하던 당시 당시 MySQL 3.X는 지금의 성숙한 MySQL 5.X 버전에 비하면
여러가지로 부족한 점이 많았습니다. - 단일 저장 engine 지원(MyISAM),
컴파일타임 전역 인코딩 지정, FK 지원, View/Trigger/SP 지원등 -
하지만 탁월한 읽기속도를 바탕으로 대중적인 PHP Application에 성공적으로
탑재되면서 그 뒤 10년간 거의 Open source 영역 DB의 대명사로 자리를잡았고
현재는 상용 DB와 견주어도 부족하지 않은 풍부한 기능을 제공하고 있습니다.

그동안 MySQL에 좋은 기능들이 많이 추가되어 유용하게 잘 사용하고 있지만
개인적으로 MySQL이라는 DB를 떠올리면 가장 먼저 떠오르는 애증의 기능이 하나
있다면 바로 오늘 소개할 `enum`이라는 자료형입니다. `enum`은 C 언어에 익숙한
분들이라면 이미 눈치 체셨겟지만 enumeration을 의미하는 자료형으로 단일 컬럼에
미리 지정한 자료만 저장 가능하도록 하는 자료형 입니다. 예를 들면
enum('a','b','c')으로 지정된 컬럼은 'a','b','c' 값만이 저장이 됩니다.

광범위하게 enum을 사용할 때 얻을 수 있는 장점은 다음과 같습니다.

 1. Text(varchar,char,text)로 값을 저장하는것에 비해 자료의 정합성을 높힐 수 있다
 2. Text + FK 으로 설계하는 것에 비해 Table수를 획기적으로 줄일 수 있다.
 3. enum index 를 통한 연산을 사용할 수 있다.

 1은 부연설명이 필요없는 장점입니다. text field column은 사실상 너무 많이 가능성이 열려있는
필드이기 때문에 아무리 application에서 정합성을 체크해도 조금만 실수하면 바로 엉뚱한 값들이
들어와 속을 썩이기 마련입니다. 2는 조금 논쟁이 있을 수 있는 장점입니다. 사실 앞서 설명한
정합성에 대한 부분을 정석으로 푸는것은 해당 컬럼에 들어올 수 있는 값을 row로 가진 data table을
생성하고 그 컬럼에서 FK 제약 조건을 통해 정합성을 확보하는 방식입니다. 이 방식은 어떤 DBMS에서
사용할수 있는 방법이기때문에 portability가 뛰어나지만 결정적으로 enum으로 지정될 만한 컬럼이
많은경우 컬럼별로 테이블이 늘어나야 하는 문제가 발생하거나 통합적인 index 테이블을 사용 한다
하더라도 여려가지 신경써야 할 문제들이 많아지기 때문에 enum은 경우에 따라서는 값싸고
훌륭한 대안입니다. 3은 의외로 모르는 분들이 많은 기능이지만 개인적으로 MySQL enum의
가장 멋진 기능이라고 생각하는 기능인데 간략하게 말하면 enum 컬럼에 입력과 출력에 있어서 enum으로
지정된 '값' 자체 뿐만 아니라 index 값을 사용할 수 있습니다. 예를 들어 다음과 같은 테이블이 있다고 할때

    +-------+-------------------+------+-----+---------+----------------+
    | Field | Type              | Null | Key | Default | Extra          |
    +-------+-------------------+------+-----+---------+----------------+
    | id    | int(11) unsigned  | NO   | PRI | NULL    | auto_increment |
    | c1    | enum('a','b','c') | YES  |     | NULL    |                |
    +-------+-------------------+------+-----+---------+----------------+

다음 쿼리의 결과는 아래와 같습니다.

    SELECT id,c1,c1+0 FROM `t1`
    +----+------+------+
    | id | c1   | c1+0 |
    +----+------+------+
    |  1 | a    |    1 |
    |  2 | b    |    2 |
    |  3 | c    |    3 |
    +----+------+------+

그후 다음 쿼리의 결과는 다음과 같습니다.

    INSERT INTO t1(c1) VALUES(1);
    SELECT id,c1,c1+0 FROM `t1`
    +----+------+------+
    | id | c1   | c1+0 |
    +----+------+------+
    |  1 | a    |    1 |
    |  2 | b    |    2 |
    |  3 | c    |    3 |
    |  4 | a    |    1 |
    +----+------+------+

이처럼 SELECT 문에서 enum 컬럼에 숫자 연산을 하면 index 값을 반환합니다. 마찬가지로 INSERT / UPDATE 문에서
enum 컬럼에 index를 넣으면 자동으로해당 컬럼을 넣어줍니다. 물론 이 enum 컬럼을 사용할때 몇가지 꼭 주의해야
할 부분이 있지만 그 부분을 제외하고는 아주 훌륭한 도구입니다. 특히 이런 특징은 웹 프로그래밍에서 HTML의
Input element 중 Radio element 와 아주 궁합이 잘 맞습니다.  예를들면 위에 설명한 c1 컬럼에 값을 넣는
Radio element 는 다음과 같이 단순하게 작성할 수 있습니다.

     <input type="radio" id="c11" name="c1" value="1"> a
     <input type="radio" id="c12" name="c1" value="2"> b
     <input type="radio" id="c13" name="c1" value="3"> c

이런 방식은 Radio element를 수백개 이상 다뤄야하는 그동안의 작업을 아주 단순하게 만들어 줬습니다.

Perl test code
--------------

이 포스트는 단순한 `enum` 컬럼의 장점 뿐만 아니라 그동안 enum 컬럼을 광범위하게 사용하면서 만났던
몇가지 문제들을 재현할 수 있는 perl test 코드를 작성해보는것을 목적으로 하고 잇습니다. 그동안 enum을
사용할 때 발생하는 부작용은 항상 문제가 발생하고 나서 뒤늦게 확인하는 경향이 있고 다루는 자료가 너무
많다 보니 문제를 재현하기 어려웠는데 이런 문제를 해결하기 위해서는 문제를 충분히 재현하고 공유할 수
있는 코드를 작성하는 편이 더 효율적이라는 결론은 내렸습니다.

코드에는 평소에 제가 즐겨쓰는 몇가지 모듈이 사용됩니다. [Test::Most][test-most], [DBIx::Simple][dbix-simple],
[SQL::Abstract][sql-abstract], [Test::DatabaseRow][test-databaserow]

 * Test::Most는 Test::More와 이름이 거의 흡사하지만 자주 사용하는 Test::Differences,Test::Deep,Test::Exception 등을
   같이 불러주기 때문에 습관적으로 사용합니다.
 * DBIx::Simple + SQL::Abstract는 복잡하게 ORM을 사용하지 않고 간편하게 DB에 값을 주고 받을때 사용합니다.
 * Test::DatabaseRow는 DB의 구조가 아닌 `값`을 기준으로 검증코드를 만들고자 할때 사용합니다. 최근에 특정작업을 진행
   하면서 많이 사용하게 되었습니다.

Code
----

``` perl
#!/usr/bin/env perl
use Test::Most;
use Test::DatabaseRow;
use Test::Group;
use DBIx::Simple;
use SQL::Abstract;
use Const::Fast;

const my $DATABASE => 'testdb';
const my $TABLE    => 't1';
const my $COLUMN   => 'c1';

const my $DBHOST => '127.0.0.1';
const my $DBUSER => 'root';
const my $DBPASS => q();


my $db = DBIx::Simple->connect(
    "DBI:mysql:host=$DBHOST",
    $DBUSER, $DBPASS,
    {
        mysql_enable_utf8    => 1,
        mysql_auto_reconnect => 1,
    }
) or die;
$db->abstract = SQL::Abstract->new( { quota_char => '`', name_sep => '.' } );

local $Test::DatabaseRow::dbh = $db->dbh;

subtest "enum column에 컬럼 추가가 발생한경우(POST)" =>  sub {
    my %h = (
        1 => { n => 'a', i => 1 },
        2 => { n => 'b', i => 2 },
        3 => { n => 'c', i => 3 },
    );

    initialize(%h);
#    +-------+-------------------+------+-----+---------+----------------+
#    | Field | Type              | Null | Key | Default | Extra          |
#    +-------+-------------------+------+-----+---------+----------------+
#    | id    | int(11) unsigned  | NO   | PRI | NULL    | auto_increment |
#    | c1    | enum('a','b','c') | YES  |     | NULL    |                |
#    +-------+-------------------+------+-----+---------+----------------+

    insert_values(%h);
    test_column_value(%h);
    test_column_index(%h);

    query("ALTER TABLE `$TABLE` CHANGE `$COLUMN` `$COLUMN` ENUM('a','b','c','x','y','z')  NULL  DEFAULT NULL");

#    +-------+-------------------------------+------+-----+---------+----------------+
#    | Field | Type                          | Null | Key | Default | Extra          |
#    +-------+-------------------------------+------+-----+---------+----------------+
#    | c1    | enum('a','b','c','x','y','z') | YES  |     | NULL    |                |
#    +-------+-------------------------------+------+-----+---------+----------------+

    test_column_value(%h);
    test_column_index(%h);
};

subtest "enum column에 컬럼 추가가 발생한경우(PRE)" =>  sub {
    my %h = (
        1 => { n => 'a', i => 1 },
        2 => { n => 'b', i => 2 },
        3 => { n => 'c', i => 3 },
    );

    initialize(%h);
#    +-------+-------------------+------+-----+---------+----------------+
#    | Field | Type              | Null | Key | Default | Extra          |
#    +-------+-------------------+------+-----+---------+----------------+
#    | id    | int(11) unsigned  | NO   | PRI | NULL    | auto_increment |
#    | c1    | enum('a','b','c') | YES  |     | NULL    |                |
#    +-------+-------------------+------+-----+---------+----------------+

    insert_values(%h);
    test_column_value(%h);
    test_column_index(%h);

    query("ALTER TABLE `$TABLE` CHANGE `$COLUMN` `$COLUMN` ENUM('x','y','z','a','b','c')  NULL  DEFAULT NULL");

#    +-------+-------------------------------+------+-----+---------+----------------+
#    | Field | Type                          | Null | Key | Default | Extra          |
#    +-------+-------------------------------+------+-----+---------+----------------+
#    | c1    | enum('x','y','z','a','b','c') | YES  |     | NULL    |                |
#    +-------+-------------------------------+------+-----+---------+----------------+

    test_column_value(%h);
    test_column_index(%h); # Fail
};

subtest "enum column에 숫자 이름의 컬럼이 들어간 경우(1,2,3)" =>  sub {
    my %h = (
        1 => { n => '1', i => 1 },
        2 => { n => '2', i => 2 },
        3 => { n => '3', i => 3 },
    );

    initialize(%h);
#    +-------+-------------------+------+-----+---------+----------------+
#    | Field | Type              | Null | Key | Default | Extra          |
#    +-------+-------------------+------+-----+---------+----------------+
#    | id    | int(11) unsigned  | NO   | PRI | NULL    | auto_increment |
#    | c1    | enum('1','2','3') | YES  |     | NULL    |                |
#    +-------+-------------------+------+-----+---------+----------------+

    insert_values(%h);
    test_column_value(%h);
    test_column_index(%h);

    query("INSERT INTO t1(id,c1) VALUES(4,1)");
    all_row_ok(
        table       => $TABLE,
        where       => [ id => 4 ],
        tests       => [ $COLUMN => 1 ],
        description => "if input just 1"
    );

    query("INSERT INTO t1(id,c1) VALUES(5,'1')");
    all_row_ok(
        table       => $TABLE,
        where       => [ id => 5 ],
        tests       => [ $COLUMN => 1 ],
        description => "if input '1'"
    );
};


subtest "enum column에 숫자 이름의 컬럼이 들어간 경우(0,1,2)" =>  sub {
    my %h = (
        1 => { n => '0', i => 1 },
        2 => { n => '1', i => 2 },
        3 => { n => '2', i => 3 },
    );

    initialize(%h);
#    +-------+-------------------+------+-----+---------+----------------+
#    | Field | Type              | Null | Key | Default | Extra          |
#    +-------+-------------------+------+-----+---------+----------------+
#    | id    | int(11) unsigned  | NO   | PRI | NULL    | auto_increment |
#    | c1    | enum('0','1','2') | YES  |     | NULL    |                |
#    +-------+-------------------+------+-----+---------+----------------+

    insert_values(%h);
    test_column_value(%h);
    test_column_index(%h); #Fail

    query("INSERT INTO t1(id,c1) VALUES(4,1)");
    all_row_ok(
        table       => $TABLE,
        where       => [ id => 4 ],
        tests       => [ $COLUMN => 1 ],
        description => "if input just 1"
    );

    query("INSERT INTO t1(id,c1) VALUES(5,'1')");
    all_row_ok(
        table       => $TABLE,
        where       => [ id => 5 ],
        tests       => [ $COLUMN => 1 ],
        description => "if input '1'"
    );
};

done_testing();

sub query($) {
    my ($query) = @_;
    $db->query($query) && print STDERR ">> ", $query, "\n";
}

sub initialize {
    my %h = @_;

    my @cols = map { sprintf qq('$h{$_}->{n}') } sort keys %h;
    my $enum = sprintf(q{ENUM(%s)},join(',',@cols) );

    print STDERR "Database : $DATABASE\n";

    query("DROP DATABASE IF EXISTS `$DATABASE`");
    query("CREATE DATABASE `$DATABASE` DEFAULT CHARACTER SET `utf8`");
    query("USE `$DATABASE`");
    query("CREATE TABLE `$TABLE` (id INT(11) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT) DEFAULT CHARACTER SET `utf8` ENGINE = `InnoDB`");
    query("ALTER TABLE `$TABLE` ADD `$COLUMN` $enum  NULL  DEFAULT NULL  AFTER `id`");

    print STDERR '-' x 80, "\n";
}

sub insert_values {
    my %h = @_;

    foreach my $k ( sort keys %h ) {
        $db->insert( $TABLE, { id => $k, $COLUMN => $h{$k}->{'n'} } );
    }
}

sub test_column_value {
    my %h = @_;

    foreach my $k ( sort keys %h ) {
        all_row_ok(
            table       => $TABLE,
            where       => [ id => $k ],
            tests       => [ $COLUMN => $h{$k}->{n} ],
            description => "row $k // $COLUMN is $h{$k}->{'n'}"
        );
    }
}

sub test_column_index {
    my %h = @_;

    foreach my $k ( sort keys %h ) {
        all_row_ok(
            sql => [ "SELECT $COLUMN + 0 as i FROM $TABLE WHERE id = ?", $k ],
            tests => [ i => $h{$k}->{'i'} ],
            description => "$k: index of $COLUMN is $h{$k}->{'i'}"
        );
    }
}
```

enum side effects
-----------------

위 테스트를 통해 알수있는 enum의 부작용은 다음과 같습니다.

 1. enum으로 정의된 컬럼에 내부 index에 영향을 주도록 컬럼 순서를 변경하면 '값'에는 영향이 없지만 'index'에는 영향이 생긴다
 2. enum의 값으로 index 와 동일한 숫자를 사용하지만 순서가 일치하지 않을경우 넘겨지는 예상하는 결과와 다르게 동작할 수 있다

이 밖에 enum 컬럼을 사용했을때 고려해야 할 사항은 다음 링크를 참고하시기 바랍니다. [8 Reasons Why MySQL's ENUM Data Type Is Evil](http://komlenic.com/244/8-reasons-why-mysqls-enum-data-type-is-evil/)

Summary
--------

1. MySQL의 enum type은 경우에 따라 강력하지만 부작용에 주의해야한다
2. Test 코드를 작성할 때 Test::Most는 편리하다
3. DB의 값을 확인할때 Test::DatabaseRow가 편리하다
