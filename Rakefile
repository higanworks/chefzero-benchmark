require 'chef_zero/server'
require 'ridley'
require 'fauxhai'
require 'benchmark'
require 'tempfile'
require 'faker'

TIMES = [50, 100, 500, 1000, 5000]

def ridley_init
  f = Tempfile.new('pem')
  File.write(f.path, OpenSSL::PKey::RSA.new(2048))

  ridley = Ridley.new(
    server_url: "http://localhost:4000",
    client_name: "reset",
    client_key: f.path
  )
  ridley.logger.level = 3
  ridley
end

def create_fake_node(ridley, count)
  count.times do |t|
    obj = ridley.node.new
    obj.name = [Faker::Internet.domain_word + t.to_s , Faker::Internet.domain_name].join('.')
    obj.chef_environment = get_env_sample
    obj.automatic = Fauxhai.mock(platform: 'ubuntu', version: '12.04').data
    ridley.node.create(obj)
  end
end

def get_env_sample
  [
    'development',
    'production',
    'office',
    'resort'
  ].sample
end

def version
  puts '============================'
  puts ['RUBY_VERSION', RUBY_VERSION].join(' #=> ')
  puts ['RbConfig::CONFIG["PATCHLEVEL"]', RbConfig::CONFIG['PATCHLEVEL']].join(' #=> ')
  puts ['ChefZero::VERSION', ChefZero::VERSION].join(' #=> ')
  puts '============================'
end

desc 'run benchmack both mem_store and disk_store.'
task :default do
  Rake::Task['bench:mem'].invoke
  Rake::Task['bench:disk'].invoke
end


desc 'memory_store benchmack'
namespace :bench do
  task :mem do
    version
    server = ChefZero::Server.new(port: 4000)
    server.start_background

    ridley = ridley_init

    TIMES.each do |time|
      puts "Preparing fake node objects upto #{time}..."
      ## Warmup
      puts Benchmark::CAPTION
      puts Benchmark.measure {
        create_fake_node(ridley, time)
      }
      puts "Done!"
      puts "============================"

      puts "Start benchmark for search 5 times by simple match query of #{time} nodes (on memory)"
      5.times do
        puts Benchmark::CAPTION
        puts Benchmark.measure {
          arr =  ridley.search(:node, "chef_environment:#{get_env_sample}")
          puts "                   Matched #{arr.size} nodes of #{time} objects"
        }
      end
      puts "Start benchmark for search 5 times with wildcard match query of #{time} nodes (on memory)"
      5.times do
        puts Benchmark::CAPTION
        puts Benchmark.measure {
          arr =  ridley.search(:node, "name:*#{Faker::Internet.domain_suffix}")
          puts "                   Matched #{arr.size} nodes of #{time} objects"
        }
      end
      puts "Done! Data will be cleared."
      puts "============================\n\n"
      server.clear_data
    end
    server.stop
  end

  desc 'disk benchmack'
  task :disk do
    require 'chef/chef_fs/chef_fs_data_store'
    require 'chef/chef_fs/config'
    require 'chef_zero/data_store/raw_file_store'

    Dir.mktmpdir do |tmpdir|
      puts "Temporary data_store was created to #{tmpdir}."
      Chef::Config[:chef_repo_path] = tmpdir
      chef_fs = Chef::ChefFS::Config.new.local_fs
      chef_fs.write_pretty_json = true
      data_store = Chef::ChefFS::ChefFSDataStore.new(chef_fs)

      server_options = {}
      server_options[:data_store] = data_store
      server_options[:port] = 4000

      server = ChefZero::Server.new(server_options)
      server.start_background

      ridley = ridley_init

      TIMES.each do |time|
        puts "Preparing fake node objects upto #{time}..."
        ## Warmup
        puts Benchmark::CAPTION
        puts Benchmark.measure {
          create_fake_node(ridley, time)
        }
        puts "Done!"
        puts "============================"

        puts "Start benchmark for search 5 times by simple match query of #{time} nodes (on filesystem)"
        5.times do
          puts Benchmark::CAPTION
          puts Benchmark.measure {
            arr =  ridley.search(:node, "chef_environment:#{get_env_sample}")
            puts "                   Matched #{arr.size} nodes of #{time} objects"
          }
        end
        puts "Start benchmark for search 5 times with wildcard match query of #{time} nodes (on filesystem)"
        5.times do
          puts Benchmark::CAPTION
          puts Benchmark.measure {
            arr =  ridley.search(:node, "name:*#{Faker::Internet.domain_suffix}")
            puts "                   Matched #{arr.size} nodes of #{time} objects"
          }
        end

        puts "Done! Data will be cleared."
        puts "============================\n\n"

        chef_fs.child_paths.each_value do |dirs|
          dirs.each do |dir|
            if File.exists?(dir)
              FileUtils.rm(Dir.glob("#{dir}/*.json"))
            end
          end
        end
        server.clear_data
      end
      server.stop
    end
  end
end

