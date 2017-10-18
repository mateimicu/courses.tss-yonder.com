require 'rake'
require 'date'
require 'yaml'
require 'html-proofer'

CONFIG = YAML.load(File.read('_config.yml'))
REPOSITORY = CONFIG["repo"]
SOURCE_BRANCH = CONFIG["source_branch"]
DESTINATION_BRANCH = CONFIG["destination_branch"]
DESTINATION = CONFIG["destination"]

def check_destination
  unless Dir.exist? CONFIG["destination"]
    sh "git clone https://$GIT_NAME:$GH_TOKEN@github.com/#{REPOSITORY}.git #{DESTINATION}"
  end
end

namespace :site do

  desc "Generate the site and push changes to remote origin"
  task :deploy do
    # Detect pull request
    if ENV['TRAVIS_PULL_REQUEST'].to_s.to_i > 0
      puts 'Pull request detected. Not proceeding with deploy.'
      exit
    end

    # Configure git if this is run in Travis CI
    if ENV["TRAVIS"]
      sh "git config --global user.name $GIT_NAME"
      sh "git config --global user.email $GIT_EMAIL"
      sh "git config --global push.default simple"
    end

    # Make sure destination folder exists as git repo
    check_destination

    sh "git checkout #{SOURCE_BRANCH}"
    Dir.chdir(DESTINATION) { sh "git checkout #{DESTINATION_BRANCH}" }

    # Generate the site
    sh "bundle exec jekyll build"

    # Commit and push to github
    sha = `git log`.match(/[a-z0-9]{40}/)[0]
    Dir.chdir(CONFIG["destination"]) do
      sh "git add --all ."
      sh "git commit -m 'Updating to #{REPOSITORY}@#{sha}.'"
      sh "git push --quiet origin #{DESTINATION_BRANCH}"
      puts "Pushed updated branch #{DESTINATION_BRANCH} to GitHub Pages"
    end
  end

  desc "Test the website"
  task :test do
    options = {
      :allow_hash_href => true,
      :check_html => true,
      :check_external_hash => false,
      :disable_external => true,
    }
    begin
      HTMLProofer.check_directory(DESTINATION, options).run
    rescue => error_message
      puts "#{error_message}"
    end
  end

end
