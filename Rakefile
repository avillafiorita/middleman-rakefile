# coding: utf-8

#
# Configuration Parameters 
#
# @deploy_cmd set command to deploy your website
#
# use the special symbol __build_dir__ to indicate the build directory.
# At runtime, __build_dir__ will be replaced by the appropriate build
# dir.
#
# If you need to deploy to different environments with different
# commands, you can create a file _env-<environment name>.rb (e.g.,
# _env-production.rb), where you can customize the variables for the
# specific environment.
#
# Example
#
#     rsync -avz __build_dir__ user@example.com:/dest/dir/
#
@deploy_cmd = "rsync -avz __build_dir__ user@example.com:/some/dir"

# git_check    if true, block deployment if the monitored remote branch is ahead of
#              the current branch.  Publishing an out-of-date
#              repository, in fact, could cause a regression on the content of the
#              website
#
# git_autopush if true, after deployment, add, commit, and change any local
#              change to the remote repository
#
# git_branch   the remote branch to monitor.  It defaults to master; gh-pages is the
#              value you want for GitHub pages.
#

@git_check ||= true
@git_autopush ||= false
@git_branch ||= "master"


#
# --- NO NEED TO TOUCH ANYTHING BELOW THIS LINE ---
#

require 'find'
require 'date'

# load additional rake tasks in lib/tasks
# (http://stackoverflow.com/questions/13762152/load-rake-files-and-run-tasks-from-other-files)
Rake.add_rakelib 'lib/tasks'

task :default => :preview

#
# Tasks start here
#

desc "Preview the website (an alias for 'middleman')"
task :preview do
  system "middleman"
end

desc "An alias for deploying to the production environment"
task :publish => :deploy

task :set_env, [:env] do |_, args|
  args.with_defaults(:env => "production")
  @env = args[:env]
  @build_dir = (@env == "production" ? "build" : "build-#{@env}")
  @last_deploy = "_deploy-#{@env}.txt"

  # load any environment specific configuration
  env_conf = "_env-#{@env}.rb"
  if File.exist?(env_conf) then
    load env_conf
    puts "Loading #{env_conf}"
  end
  @deploy_cmd.gsub!("__build_dir__", '"' + @build_dir + '"')

  puts "Rakefile environment:"
  puts "- environment:  #{@env}"
  puts "- output dir:   #{@build_dir}"
  puts "- deploy cmd:   #{@deploy_cmd}"
  puts "- git check:    #{@git_check}"
  puts "- git autopush: #{@git_autopush}"
  puts "- git branch:   #{@git_branch}"
  puts ""
end

desc "Build, if source or data is newer than the build dir"
task :build, [:env] => :set_env do |_, args|
  printf "Checking if I need to build ... "
  newest_source = newest_in "source"
  newest_data   = newest_in "data"
  newest_build  = newest_in @build_dir

  if newest_source[1] > newest_build[1] or newest_data[1] > newest_build[1]
    printf "yes!\n"
    build @env, @build_dir
  else
    printf "no!\n"
    puts "build directory '#{@build_dir}' is newer than source."
    puts "Use 'rake force_build #{@env}' to force a build."
  end
end

desc "Build!"
task :force_build, [:env] => :set_env do |_, args|
  build
end

desc "Deploy, if needed"
task :deploy, [:env] => :build do |_, args|
  printf "Checking if I need to deploy ... "

  newest_build = newest_in @build_dir
  if not File.exists?(@last_deploy) or File.new(@last_deploy).mtime < newest_build[1]
    printf "yes!\n"

    if git_requires_attention(@git_branch) then
      puts "\n\nWarning! It seems that the local repository is not in sync with the remote.\n"
      puts "This could be ok if the local version is more recent than the remote repository.\n"
      puts "Deploying before committing might cause a regression of the website (at this or the next deploy).\n\n"
      puts "Are you sure you want to continue? [Y|n]"

      ans = STDIN.gets.chomp
      exit if ans != 'Y' 
    end
    
    %x{git add -A && git commit -m "autopush by Rakefile at #{time}" && git push} if @git_autopush
    deploy_cmd
  else
    printf "no.\n"
    puts "#{@last_deploy} is newer than build.  Use force_deploy to force a deploy."
  end
end

desc "Deploy!"
task :force_deploy, [:env] => :set_env do |_, args|
  deploy_cmd
end

desc "List the file changed since last deploy"
task :list_changes, [:env] => :set_env do |t, args|
  content = file_changed @last_deploy
  puts content.join("\n")
end

desc "Check timestamps"
task :check_timestamps, [:env] => :set_env do
  results = []
  puts "Newest files per directory: "
  (["source", "data"] + Dir.glob("_last_deploy*") + Dir.glob("build*")).each do |file|
    results << [file] + newest_in(file)
  end
  sorted = results.sort {|x,y| y[2] <=> x[2]}
  sorted.map { |x| puts "%-20s%-50s%10s" % [x[0], x[1], x[2].strftime("%B %d @ %H:%M:%S")] }
  puts ""
  puts "The newest file of the bunch is: " + newest.first[0]
end


#
# SUPPORT FUNCTIONS
#

def build
  system "bundle exec middleman build --build-dir=#{@build_dir} -e #{@env}"
end

def deploy_cmd
  system @deploy_cmd
  system "echo $(date) > #{@last_deploy}"
end

#
# check the newest file in a directory
#

# return the newest file in a directory or a special token
def newest_in dir
  File.exists?(dir) ? newest_file_in(dir) : ["not existent", Time.new("1/1/1970")]
end

# return the newest element in dir as a pair [name, mtime]
def newest_file_in dir
  Find.find(dir).map { |x| [x, File.new(x).mtime] }.sort { |x,y| y[1] <=> x[1] }.first
end

# return an array of the files changed in source since _last_deploy
def file_changed last_deploy
  content = []
  IO.popen('find source data -newer #{last_deploy} -type f') do |io| 
    while (line = io.gets) do
      filename = line.chomp
      if user_visible(filename) then
        content << filename
      end
    end
  end 
  content
end

# this is the list of files we do not want to show in changed files
EXCLUSION_LIST = [/.*~/, /^_.*/, ".DS_Store", "javascripts?", "js", /stylesheets?/, "css", /s[ca]ss/, /.*\.css/, /.*\.js/]

# return true if filename will most likely be visible to the user on
# the published website (e.g., it is not javascript, css, ...)
def user_visible(filename)
  exclusion_list = Regexp.union(EXCLUSION_LIST)
  # the file is visible if it is not in the EXCLUSION_LIST (pardon the double negation)
  not filename.match(exclusion_list)
end 

#
# integration with git
#

def git_local_diffs
  %x{git diff --name-only} != ""
end

def git_remote_diffs branch
  %x{git fetch}
  %x{git rev-parse #{branch}} != %x{git rev-parse origin/#{branch}}
end

def git_repo?
  %x{git status} != ""
end

def git_requires_attention branch
  @git_check and git_repo? and git_remote_diffs(branch)
end

