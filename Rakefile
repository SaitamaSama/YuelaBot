require 'rake'
require 'erb'
require 'rake/testtask'
require 'active_record'
require 'dotenv/load'

include ActiveRecord::Tasks
DatabaseTasks.database_configuration = YAML.load(ERB.new(File.read('./config/database.yml')).result)
DatabaseTasks.root = "."
DatabaseTasks.db_dir = 'db'
DatabaseTasks.env = ENV['RACK_ENV'] || 'development'
DatabaseTasks.migrations_paths = ["./db/migrate"]

task :environment do
  ActiveRecord::Base.configurations = DatabaseTasks.database_configuration
  ActiveRecord::Base.establish_connection DatabaseTasks.env.to_sym
end

load 'active_record/railties/databases.rake'

namespace :g do
  desc "Generate migration"
  task :migration do
    name = ARGV[1] || raise("Specify name: rake g:migration your_migration")
    timestamp = Time.now.strftime("%Y%m%d%H%M%S")
    path = File.expand_path("../db/migrate/#{timestamp}_#{name}.rb", __FILE__)
    migration_class = name.split("_").map(&:capitalize).join

    File.open(path, 'w') do |file|
      file.write <<-EOF
class #{migration_class} < ActiveRecord::Migration[5.2]
  def change
  end
end
      EOF
    end

    puts "Migration #{path} created"
    abort # needed stop other tasks
  end
end

task :console do
  require 'require_all'
  require 'irb'
  require_relative 'models/application_record'
  require_all './models'
  ActiveRecord::Base.configurations = DatabaseTasks.database_configuration
  ActiveRecord::Base.establish_connection DatabaseTasks.env.to_sym
  ARGV.clear
  IRB.start
end

task :deploy do
  require 'net/scp'
  require 'net/ssh'

  host = "atlas.hostineer.com"
  tar = "yuelabot.tar.gz"

  `tar -czf ../#{tar} .`
  Net::SCP.upload!(
      host,
      ENV['DEPLOY_USER'],
      "../yuelabot.tar.gz",
      "yuelabot.tar.gz",
      ssh: { password: ENV['DEPLOY_PASS'] }
  )
  Net::SSH.start(host, ENV['DEPLOY_USER'], password: ENV['DEPLOY_PASS']) do |ssh|
    ssh.exec! "mkdir -p ~/yuelabot"
    ssh.exec! "tar -C ~/yuelabot/ -zxvf #{tar}"
    ssh.exec! "cd yuelabot"
    p ssh.exec! "rvm do 2.4.1 bundle install"
    p ssh.exec! "rvm do 2.4.1 rake db:migrate"
    p ssh.exec! "rvm do 2.4.1 god restart yuela"
  end
end


Rake::TestTask.new do |t|
  t.libs << "test"
  t.test_files = FileList['test/*_test.rb']
end

task default: :test