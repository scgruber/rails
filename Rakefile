require 'rake'

env = %(PKG_BUILD="#{ENV['PKG_BUILD']}") if ENV['PKG_BUILD']

PROJECTS = %w(activesupport railties actionpack actionmailer activeresource activerecord)

Dir["#{File.dirname(__FILE__)}/*/lib/*/version.rb"].each do |version_path|
  require version_path
end

desc 'Run all tests by default'
task :default => :test

%w(test rdoc pgem package release gem).each do |task_name|
  desc "Run #{task_name} task for all projects"
  task task_name do
    PROJECTS.each do |project|
      warn 'When running tests for Rails LTS, prefer running the "railslts:test" task.'
      system %(cd #{project} && #{env} rake _#{RAKEVERSION}_ #{task_name}) or raise 'failed'
    end
  end
end

if RAKEVERSION == '0.8.0'
  require 'rake/rdoctask'

  # In order to generate the API please install Ruby 1.8.7 and rake 0.8.0. Then:
  #
  #     rake _0.8.0_ task_name
  #
  # Reason is the Jamis template used for the API needs RDoc 1.x. Recent rake libs
  # do not provide rake/rdoctask, and RDoc 1.x provides no alternative task. This
  # is easy to setup with a Ruby version manager.
  desc "Generate documentation for the Rails framework"
  Rake::RDocTask.new do |rdoc|
    rdoc.rdoc_dir = 'doc/rdoc'
    rdoc.title    = "Ruby on Rails Documentation"
    rdoc.main     = "railties/README"

    rdoc.options << '--line-numbers' << '--inline-source'
    rdoc.options << '-A cattr_accessor=object'
    rdoc.options << '--charset' << 'utf-8'
    rdoc.options << '--main' << 'railties/README'

    rdoc.template = ENV['template'] ? "#{ENV['template']}.rb" : './doc/template/horo'

    rdoc.rdoc_files.include('railties/CHANGELOG')
    rdoc.rdoc_files.include('railties/LICENSE')
    rdoc.rdoc_files.include('railties/README')
    rdoc.rdoc_files.include('railties/lib/{*.rb,commands/*.rb,rails/*.rb,rails_generator/*.rb}')

    rdoc.rdoc_files.include('activerecord/README')
    rdoc.rdoc_files.include('activerecord/CHANGELOG')
    rdoc.rdoc_files.include('activerecord/lib/active_record/**/*.rb')
    rdoc.rdoc_files.exclude('activerecord/lib/active_record/vendor/*')

    rdoc.rdoc_files.include('activeresource/README')
    rdoc.rdoc_files.include('activeresource/CHANGELOG')
    rdoc.rdoc_files.include('activeresource/lib/active_resource.rb')
    rdoc.rdoc_files.include('activeresource/lib/active_resource/*')

    rdoc.rdoc_files.include('actionpack/README')
    rdoc.rdoc_files.include('actionpack/CHANGELOG')
    rdoc.rdoc_files.include('actionpack/lib/action_controller/**/*.rb')
    rdoc.rdoc_files.include('actionpack/lib/action_view/**/*.rb')
    rdoc.rdoc_files.exclude('actionpack/lib/action_controller/vendor/*')

    rdoc.rdoc_files.include('actionmailer/README')
    rdoc.rdoc_files.include('actionmailer/CHANGELOG')
    rdoc.rdoc_files.include('actionmailer/lib/action_mailer/base.rb')
    rdoc.rdoc_files.exclude('actionmailer/lib/action_mailer/vendor/*')

    rdoc.rdoc_files.include('activesupport/README')
    rdoc.rdoc_files.include('activesupport/CHANGELOG')
    rdoc.rdoc_files.include('activesupport/lib/active_support/**/*.rb')
    rdoc.rdoc_files.exclude('activesupport/lib/active_support/vendor/*')
  end

  # Enhance rdoc task to copy referenced images also
  task :rdoc do
    FileUtils.mkdir_p "doc/rdoc/files/examples/"
    FileUtils.copy "activerecord/examples/associations.png", "doc/rdoc/files/examples/associations.png"
  end
end

namespace :railslts do

  desc 'Run tests for Rails LTS compatibility'
  task :test do

    puts '', "\033[44m#{'activesupport'}\033[0m", ''
    system('cd activesupport && rake test')

    puts '', "\033[44m#{'actionmailer'}\033[0m", ''
    system('cd actionmailer && rake test')

    puts '', "\033[44m#{'actionpack'}\033[0m", ''
    system('cd actionpack && rake test')

    puts '', "\033[44m#{'activerecord (mysql)'}\033[0m", ''
    system('cd activerecord && rake test_mysql')

    puts '', "\033[44m#{'activerecord (sqlite3)'}\033[0m", ''
    system('cd activerecord && rake test_sqlite3')

    puts '', "\033[44m#{'activerecord (postgres)'}\033[0m", ''
    system('cd activerecord && rake test_postgresql')

    puts '', "\033[44m#{'activeresource'}\033[0m", ''
    system('cd activeresource && rake test')

    puts '', "\033[44m#{'railties'}\033[0m", ''
    system('cd railties && rake test')

  end

  task :clean_gems do
    PROJECTS.each do |project|
      pkg_folder = "#{project}/pkg"
      puts "Emptying packages folder #{pkg_folder}..."
      FileUtils.mkdir_p(pkg_folder)
      system("rm -rf #{pkg_folder}/*") or raise "failed"
    end
  end

  task :clean_building_artifacts do
    PROJECTS.each do |project|
      pkg_folder = "#{project}/pkg"
      puts "Deleting building artifacts from #{pkg_folder}..."
      system("rm -rf #{pkg_folder}/*.tgz") or raise "failed" # TGZ
      system("rm -rf #{pkg_folder}/*.zip") or raise "failed" # ZIP
      system("rm -rf #{pkg_folder}/*/") or raise "failed"    # Folder
    end
  end

  task :zip_gems do
    puts "Zipping archive for manual installation..."
    archive_name = "railslts.tar.gz"
    system("cd dist && rm -f #{archive_name} && tar -czvhf #{archive_name} railslts/ && cd ..") or raise "failed"
  end

  desc 'Builds *.gem packages for static distribution without Git'
  task :build_gems => [:clean_gems, :package, :clean_building_artifacts, :zip_gems] do
    puts "Done."
  end

  desc 'Updates the LICENSE file in individual sub-projects'
  task :update_license do
    require 'date'
    last_change = Date.parse(`git log -1 --format=%cd`)
    PROJECTS.each do |project|
      license_path = "#{project}/LICENSE"
      puts "Updating license #{license_path}..."
      File.exists?(license_path) or raise "Could not find license: #{license_path}"
      license = File.read(license_path)
      license.sub!(/ before(.*?)\./ , " before #{(last_change + 10).strftime("%B %d, %Y")}.") or raise "Couldn't find timestamp."
      File.open(license_path, "w") { |w| w.write(license) }
    end
  end

  namespace :release do

    task :ensure_ready do
      jobs = [
        'Are you aware that you need to perform the following tasks on EVERY Rails LTS branch that was changed?',
        'Did you release a new version of https://github.com/makandra/railslts-version ?',
        'Did you bump the required "railslts-version" dependency in railties.gemspec?',
        'Did you update the LICENSE files using `rake railslts:update_license` (only for 2.3 and 3.0)??',
        'Did you build static gems using `rake railslts:build_gems` (only for 2.3 and 3.0)?',
        'Did you commit and push your changes, as well as the changes by the Rake tasks mentioned above?',
        'Did you activate key forwarding for *.gems.makandra.de?',
        'We will now publish the latest versions (2.3, 3.0 and 3.2) to gems.makandra.de. Ready?',
      ]

      puts

      jobs.each do |job|
        print "#{job} [y/n] "
        answer = STDIN.gets
        puts
        unless answer.strip == 'y'
          $stderr.puts "Aborting. Nothing was released."
          puts
          exit
        end
      end
    end


    desc "Publish new Rails LTS customer release on gems.makandra.de/railslts"
    task :customer => :ensure_ready do
      for hostname in %w[c23 c42 c32 c24]
        fqdn = "#{hostname}.gems.makandra.de"
        puts "\033[1mUpdating #{fqdn}...\033[0m"
        command = '/opt/update_railslts.sh'
        system "ssh deploy-gems_p@#{fqdn} '#{command}'" or raise 'Deployment failed'
        puts "done."
      end

      puts 'Deployment done.'
      puts "Now do a 'git clone https://gems.makandra.de/railslts lts-test-checkout',"
      puts 'switch to the right branch(es) (e.g. 3-0-lts) and make sure your commits are present.'
    end

    desc "Publish new Rails LTS community release on github.com/makandra/rails"
    task :community do
      existing_remotes = `git remote`
      unless existing_remotes.include?('community')
        system('git remote add community git@github.com:makandra/rails.git') or raise "Couldn't add remote'"
      end
      system('git fetch community') or raise 'Could not fetch from remote'

      puts "We will now publish the following changes to GitHub:"
      puts
      system('git log --oneline community/2-3-lts..HEAD') or raise 'Error showing log'
      puts

      puts "Do you want to proceed? [y/n]"
      answer = STDIN.gets
      puts
      unless answer.strip == 'y'
        $stderr.puts "Aborting. Nothing was released."
        puts
        exit
      end

      system('git push community 2-3-lts') or raise 'Error while publishing'
      puts "Deployment done."
      puts "Check https://github.com/makandra/rails/tree/2-3-lts"
    end

  end

end


