---
title: Rails ActivRecord源码（未完成）
date: 2017-04-20
tags: rails, sourcecode
---

rails activerecord 源码
--------------

## 数据库的连接调用过程

  1. activercord base中的代码

      ``` ruby
        # activerecord/lib/active_record/railtie.rb
          ActiveSupport.on_load(:active_record) do
            self.configurations = Rails.application.config.database_configuration

            begin
              establish_connection
            end
          end

        # activerecord/lib/base.rb
        class Base
          extend ConnectionHandling
          extend Core
        end

        # activerecord/lib/active_record/core.rb
        module ActiveRecord
          module Core
            extend ActiveSupport::Concern
            included do
              def self.configurations=(config)
                @@configurations = ActiveRecord::ConnectionHandling::MergeAndResolveDefaultUrlConfig.new(config).resolve
              end
            end
          end
        end

        # activerecord/lib/active_record/connectioin_handing.rb
        module ActiveRecord
          module ConnectionHandling
            def establish_connection
            end

            class MergeAndResolveDefaultUrlConfig
              def resolve
                ConnectionAdapters::ConnectionSpecification::Resolver.new(config).resolve_all
              end
            end
          end
        end

        # activerecord/lib/active_record/connectioin_adapter/connection_specification.rb
        module ActiveRecord
          module ConnectionAdapters
            class ConnectionSpecification #:nodoc:
              class Resolver
                def resolve(config)
                  if config
                    resolve_connection config
                  elsif env = ActiveRecord::ConnectionHandling::RAILS_ENV.call
                    resolve_symbol_connection env.to_sym
                  else
                    raise AdapterNotSpecified
                  end
                end

                # Expands each key in @configurations hash into fully resolved hash
                def resolve_all
                  config = configurations.dup
                  config.each do |key, value|
                    config[key] = resolve(value) if value
                  end
                  config
                end
              end
            end
          end
        end
      ```

  > 调用链从上至下， base 中extend ConnectionHandlere, Core, 在core中通过configration=函数， 将database.yml 中的数据传递给 ConnectionHandling 进行配置的解析，合并等，　降低到Resolver中， 为具体实现的策略、算法。然后再代用establish_connection 调用connnection_handling文件中的 establish_connection 的方法, 通过connection_handler.establish_connection 建立数据库连接。 到此数据库建立连接成功。

  **在查看代码的时候需要连续的挑选文件， 查找函数， 而没有统一的累死java中interface中的概念，导致看起来，module之间的连接非常脆弱， 不是显而易见。**


  2. active record ConnectionSpecification

    1. 调用方法

## 数据库迁移

  1. 调用链

      ```
        |- task :migrate
          |- DatabaseTask.migrate
            |- Migrator.migrate(migrations_paths, version || nil ) {|migration| scope.blank? ||| scope == migration.scope}
              |- Migrator.migrate(migrations_paths, target_version, &block)
                |- Migrator.up | Migrator.down
                  |- new(:up | :down, migrations, target_version).migrate
                    |- migrate_without_lock
                      |- runnable.each {|migration| execute_migration_in_transaction(migration, direction)} # runnable 寻找没有执行的migration
                        |- execute_migration_in_transaction(migrations)
                          |- migration.migrate(direction) # migration -> MigrationProxy
                            |- Migration.instance.migrate(direction)
                              |- exec_migration(conn, direction)
                                |- send(:direction) # 因为 AddColumnToVariants < ActiveRecord::Migration
                                  |- add_column :variants, :part_number, :string
                                    |- method_missing
                                      |- connection.send(:add_column, [:variants, :part_number, :string], &block)
                                        |- ActiveRecord::ConnectionHandling::ConnectionAdapters::SQLite3Adapter

       ```
  2. 总结一下流的顺序

      ```
        rake --> Migrator.migrate -> up/down ---> new(direction, migrations).migrate --> runnable.each {|item| execute_migration_in_transaction(item)} -> migration_proxy.migrate(direction) -> Migration.instance.migrate -> Migration.instance.exec_migration() -> 调用迁移文件的up/down/change ---> 泗洪method_missing hook add_column, remove_column 等发送到connection, connection 为各种数据库adapter
      ```

      > 借鉴的地方， 1. 使用MigrationProxy代替真是的文件， 比较轻，可以实现一些version，时间戳等比较方法 , 实现将文件系统转换为ruby对象， 接入到系统内部。2. 使用method_missing 来进行委托，将数据库connection中具有的方法实现到 Migration

  3. 代码

        ```ruby
          def add_column(table_name, column_name, type, options = {}) #:nodoc:
            if valid_alter_table_type?(type)
              super(table_name, column_name, type, options)
            else
              alter_table(table_name) do |definition|
                definition.column(column_name, type, options)
              end
            end
          end

          def alter_table(table_name, options = {}) #:nodoc:
            altered_table_name = "a#{table_name}"
            caller = lambda {|definition| yield definition if block_given?}

            transaction do
              move_table(table_name, altered_table_name,
                options.merge(:temporary => true))
              move_table(altered_table_name, table_name, &caller)
            end
          end

          def move_table(from, to, options = {}, &block) #:nodoc:
            copy_table(from, to, options, &block)
            drop_table(from)
          end

          def copy_table(from, to, options = {}) #:nodoc:
            from_primary_key = primary_key(from)
            options[:id] = false
            create_table(to, options) do |definition|
              @definition = definition
              @definition.primary_key(from_primary_key) if from_primary_key.present?
              columns(from).each do |column|
                column_name = options[:rename] ?
                  (options[:rename][column.name] ||
                   options[:rename][column.name.to_sym] ||
                   column.name) : column.name
                next if column_name == from_primary_key

                @definition.column(column_name, column.type,
                  :limit => column.limit, :default => column.default,
                  :precision => column.precision, :scale => column.scale,
                  :null => column.null, collation: column.collation)
              end
              yield @definition if block_given?
            end
            copy_table_indexes(from, to, options[:rename] || {})
            copy_table_contents(from, to,
              @definition.columns.map(&:name),
              options[:rename] || {})
          end

          #其中的方式处理的比较奇怪，  是copy之后然后 drop, copy现有表 到中间表 然后 删除现有表，最后copy 中间表到 现有表表名， 其中每个步骤都涉及到 copy content, copy index, copy table schema 等, 其中会涉及到性能的问题吗？比如大量的数据插入？？？？？
        ```
