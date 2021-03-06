---
layout: post
title: "dsfs 23 databases and sql"
description: ""
category: [python, data science]
tags: [python, data science, dtabase, sql]
---
{% include JB/setup %}


### databases and sql

1. create table and insert

    1. users data set user_id, name, num_friends

            users = [[0, "Hero", 0],
            [1, "Dunn", 2],
            [2, "Sue", 3],
            [3, "Chi", 3]]

            create table users (
                user_id int not null,
                name varchar(200),
                num_friends int);

            insert into users(user_id, name, num_friends) values (0, 'Hero', 0)

    1. define a class

            # a real database would never
            # use subh a space-wasting representation
            class Table:
                def __init__(self, columns):
                    self.columns = columns
                    self.rows = []

                def __repr__(self):
                    """pretty representation of the table: columns then rows"""
                    return str(self.columns) + '\n' + '\n'.join(map(str, self.rows))

                def insert(self, row_values):
                    if len(row_values) != len(self.columns):
                        raise TypeError('wrong number of elements')
                    row_dict = dict(zip(self.columns, row_values))
                    self.rows.append(row_dict)

    1. e.g.

            users = Table(["user_id", "name", "num_friends"])
            users.insert([0, "Hero", 0])
            users.insert([1, "Dunn", 2])
            users.insert([2, "Sue", 3])
            users.insert([3, "Chi", 3])
            users.insert([4, "Thor", 3])
            users.insert([5, "Clive", 2])
            users.insert([6, "Hicks", 3])
            users.insert([7, "Devin", 2])
            users.insert([8, "Kate", 2])
            users.insert([9, "Klein", 3])
            users.insert([10, "Jen", 1])

            # print users
            ['user_id', 'name', 'num_friends']
            {'user_id': 0, 'name': 'Hero', 'num_friends': 0}
            {'user_id': 1, 'name': 'Dunn', 'num_friends': 2}
            {'user_id': 2, 'name': 'Sue', 'num_friends': 3}
            ...

1. update

    1. sql

            update users set num_friends=3 where user_id=1;
            # what table to update            => users
            # what rows to update             => num_friends=3
            # which fields to update          => num_friends
            # what their new values should be => 3

    1. define a method

            def update(self, updates, predicate):
                for row in self.rows:
                    if predicate(row):
                        for column, new_value in updates.iteritem():
                            row[column] = new_value

    1. e.g.

            users.update({'num_friends': 3},
                         lambda row: row['user_id'] == 1)

1. delete

    1. sql

            delete from users;
            delete from users where user_id=1;

    1. define a method

            def delete(self, predicate=lambda row: True):
                """delete all rows matching predicate
                or all rows if no predicate supplied"""
                self.rows = [row for row in self.rows if not(predicate(row))]

    1. e.g.

            users.delete(lambda row: row['user_id'] == 1)
            users.delete()

1. select

    1. sql

            select * from users;
            select * from users limit 2;
            select user_id from users;
            select user_id from users where name='Dunn';

            select length(name) as name_length from users;

    1. define a method

            # accepts two optional arguments
            # 1. keep_columns
            # 2. additional_columns

            def select(self, keep_columns, additional_columns):
                if keep_columns is None:
                    keep_columns = self.columns

                if additional_columns is None:
                    additional_columns = {}

                result_table = Table(keep_columns + additional_columns.keys())

                for row in self.rows():
                    new_row = [row[column] for column in keep_columns]
                    for column_name, calculation in additional_columns.iteritem():
                        new_value.append(calculation(row))
                    result_table.insert(new_row)
                return result_table

            def where(self, predicate=lambda row: True):
                """return only the rows that satisfy the supplied predicate"""
                where_table = Table(self.columns)
                where_table.rows = filter(predicate, self.rows)
                return where_table

            def limit(self, num_rows):
                """return only the first num_rows rows"""
                limit_table = Table(self.columns)
                limit_table.rows = self.rows[:num_rows]
                return limit_table

    1. e.g.

            # select * from users;
            users.select()

            # select * from users limit 2;
            users.limit(2)

            # select user_id from users;
            users.select(keep_columns=['user_id'])

            # select user_id from users where name='Dunn';
            users.where(lambda row: row['name'] == 'Dunn']) \
                 .select(keep_columns=['user_id'])

            # select length(name) as name_length from users
            def name_length(row):
                return len(row['name'])

            users.select(keep_columns=[],
                         additional_columns={'name_length': name_length})

1. group by

    1. sql

            select length(name) as name_length,
                min(user_id) as min_user_id,
                count(*) as num_users
            from users
            group by length(name);

            # having => filter is applied to the aggregates
            # where  => filter out rows before aggregation even took place

            select substr(name, 1, 1) as first_letter,
                avg(num_friends) as avg_num_frientds
            from users
            group by substr(name, 1, 1)
            having avg(num_friends) > 1;

            # compute overall aggregates
            select sum(user_id) as user_id_sum
            from users
            where user_id > 1;

    1. define a method

            # arguments
            # 1. columns names to group by
            # 2. dict of aggregation functions run over each group
            # 3. optional predicate having

            # steps
            # 1. create defaultdict to map tuples (of the group-by-values) to rows (containing the group-by-values)
            #   can't use lists as dict keys, have to use tuples
            # 2. iterates over the rows of the table, populating the defaultdict
            # 3. create a new table with the correct output columns
            # 4. iterates over the defaultdict and populates the output table, applying the having filter if any

            # an actual database would almost certainly
            # do this in a more-efficient manner

            def group_by(self, group_by_columns, aggregates, having=None):
                grouped_rows = defaultdict(list)

                # populate groups
                for row in self.rows:
                    key = tuple(row[column] for column in group_by_columns)
                    grouped_rows[key].append(row)

                # result table consists of group_by columns and aggregates
                result_table = Table(group_by_columns + aggregates.keys())

                for key, rows in grouped_rows.iteritems():
                    if having is None or having(rows):
                        new_row = list(key)
                        for aggregate_name, aggregate_fn in aggregates.iteritems():
                            new_row.append(aggregate_fn(rows))
                        result_table.insert(new_row)

                return result_table

    1. e.g.

            # name_length metrics
            def min_user_id(rows):
                return min(row['user_id'] for row in rows)

            stats_by_length = users \
                .select(additional_columns={'name_length': name_length}) \
                .group_by(group_by_columns=['name_length'],
                          aggregates={'min_user_id': min_user_id,
                                      'num_users': len})

            # first_letter metrics
            def first_letter_of_name(row):
                return row['name'][0] if row['name'] / len(rows)

            def average_num_friends(rows):
                return sum(row['num_friends'] for row in rows) / len(ros)

            def enough_friends(rows):
                return average_num_friends(rows) > 1

            avg_friends_by_letter = users \
                .select(additional_columns={'first_letter': first_letter_of_name}) \
                .group_by(group_by_columns=['first_letter'], aggregates={'avg_num_friends': average_num_friends}, having=enough_friends)

            # user_id_sum
            def sum_user_ids(rows):
                return sum(row['user_id'] for row in rows)

            user_id_sum = users \
                .where(lambda row: row['user_id'] > 1) \
                .group_by(group_by_columns=[], aggregates={'user_id_sum': sum_user_ids})

1. order by

    1. sql

            select * from users order by name limit 2

    1. define a method

            def order_by(self, order):
                new_table = self.select()
                new_table.rows.sort(key=order)
                return new_table

    1. e.g.
    
            friendliest_letters = avg_friends_by_letter \
                .order_by(lambda row: -row['avg_num_friends']) \
                .limit(4)   

1. join

    1. sql

            # one-to-many relationship
            create table user_interests (
                user_id int not null,
                interest varchar(100) not null
            );

            # create the table
            user_interests = Table(['user_id', 'interest'])
            user_interests.insert([0, "SQL"])
            user_interests.insert([0, "NoSQL"])
            user_interests.insert([2, "SQL"])
            user_interests.insert([2, "MySQL"])

            # query inner join
            select users.name from users
            join user_interests
            on users.user_id = user_interests.user_id
            where user_interests.interest = 'SQL'

            # query left join
            select users.id, count(user_interests.interest) as num_interests
            from user
            left join user_interests
            on users.user_id = user_interests.user_id

    1. define a method

            def join(self, other_table, left_join=False):
                join_on_columns = [c for c in self.columns
                                   if c in other_table.columns]

                additional_columns = [c for c in other_table.columns
                                      if c not in join_on_columns]

                # all columns from left table
                # + additional_columns from right table
                join_table = Table(self.columns + additional_columns)

                for row in self.rows:
                    def is_join(other_row):
                        return all(other_row[c] == row[c] for c in join_on_columns)

                    other_rows = other_table.where(is_join).rows

                    # if no rows match and it's a left join
                    # output with Nones
                    if left_join and not other_rows:
                        join_table.insert([row[c] for c in self.columns] + [None for c in additional_columns])

                return join_table

    1. e.g.

            sql_user = users \
                .join(user_interests) \
                .where(lambda row: row['interest' == 'SQL']) \
                .select(keep_columns=['name'])

            # get interest counts with
            def count_interest(rows):
                """counts how many rows have non-None interests"""
                return len([row for row in rows
                            if row['interest'] is not None])

            user_interest_counts = users \
                .join(user_interests, left_join=True) \
                .group_by(group_by_columns=['user_id'],
                          aggregates={'num_interests': count_interest})

1. subqueries

    1. sql

            select min(user_id) as min_user_id from
            (select user_id from user_interests where interest='SQL') sql_interests;

    1. get this for free

            likes_sql_user_ids = user_interests \
                .where(lambda row: row['interest'] == 'SQL') \
                .select(keep_columns=['user_id'])

            likes_sql_user_ids.group_by(group_by_columns=[],
                                        additional_columns={'min_user_id': min_user_id})

1. indexes

1. query optimization

    1. sql

            select user.name
            from users
            join user_interests
            on users.user_id = user_interests.user_id
            where user_interests.interest = 'SQL'

    1. query 1

            # filter before join
            # is almost certainly more efficient
            user_interests \
                .where(lambda row: row['interest'] == 'SQL') \
                .join(users) \
                .select(['name'])

    1. query 2

            user_interests \
                .join(users) \
                .where(lambda row: row['interest'] == 'SQL') \
                .select(['name'])

1. nosql