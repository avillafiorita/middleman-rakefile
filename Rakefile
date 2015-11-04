# coding: utf-8

require 'find'
task :default => :preview

#
# Configuration Parameters 
#
# git_check    if true, block deployment if the default remote branch is ahead of
#              the current branch this is because publishing an out-of-date
#              repository could cause a regression on the content of the
#              website
#
# git_autopush if true, after deployment, add, commit and change any local
#              change to the remote repository
#

$git_check ||= true
$git_autopush ||= false

#
# --- NO NEED TO TOUCH ANYTHING BELOW THIS LINE ---
#

#
# Tasks start here
#

desc "Preview the website (an alias for 'middleman')"
task :preview do
  system "middleman"
end

desc "Build, if source or data is newer than the build dir"
task :build do
  printf "Checking if I need to build ... "
  newest_source = newest_file "source"
  newest_data   = newest_file "data"
  newest_build  = File.exists?("build") ? newest_file("build") : ["not existent", Date.parse("1/1/1970")]

  if newest_source[1] > newest_build[1] or newest_data[1] > newest_build[1]
    printf "yes!\n"
    system "bundle exec middleman build"
  else
    printf "no!\n"
    puts "build directory is newer than source.  Use force_build to force a build."
  end
end

desc "Build!"
task :force_build do
  system "bundle exec middleman build"
end

desc "An alias for deploy"
task :publish => :deploy

desc "Deploy, if the newest file on build is newer than _last_deploy.txt" 
task :deploy => :build do
  printf "Checking if I need to deploy ... "
  newest_build = newest_file("build") # build exists, since this task depends on "build"

  if not File.exists?("_last_deploy.txt") or File.new("_last_deploy.txt").mtime < newest_build[1]
    printf "yes!\n"

    if git_requires_attention("master") then
      puts "\n\nWarning! It seems that the local repository is not in sync with the remote.\n"
      puts "This could be ok if the local version is more recent than the remote repository.\n"
      puts "Deploying before committing might cause a regression of the website (at this or the next deploy).\n\n"
      puts "Are you sure you want to continue? [Y|n]"

      ans = STDIN.gets.chomp
      exit if ans != 'Y' 
    end
    system "bundle exec middleman deploy"
    system "echo $(date) > _last_deploy.txt"
    %x{git add -A && git commit -m "autopush by Rakefile at #{time}" && git push} if $git_autopush
  else
    printf "no.\n"
    puts "_last_deploy.txt is newer than build.  Use force_deploy to force a deploy."
  end
end

desc "Deploy!"
task :force_deploy do
  system "bundle exec middleman deploy"
  system "echo $(date) > _last_deploy.txt"
end

desc "Check timestamps"
task :check_timestamps do
  results = []
  puts "Newest files: "
  ["source", "data", "build", "_last_deploy.txt"].each do |file|
    if File.exists? file
      result = newest_file file
      results << result
      puts "%20s" % result[0] + " => " + result[1].to_s
    else
      puts "%20s" % file + " => does not exist"
    end
  end
  newest = results.sort {|x,y| y[1] <=> x[1]}.first
  puts "The newest file of the bunch is: " + newest[0]
end

desc "List the file changed since last deploy"
task :list_changes do |t, args|
  content = file_changed
  puts content.join("\n")
end

#
# SUPPORT FUNCTIONS
#

# return the newest element in dir as a pair [name, mtime]
def newest_file dir
  Find.find(dir).map { |x| [x, File.new(x).mtime] }.sort { |x,y| y[1] <=> x[1] }.first
end

# return an array of the files changed in source since _last_deploy
def file_changed
  content = []
  IO.popen('find source -newer _last_deploy.txt -type f') do |io| 
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


# integration with git

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
  $git_check and git_repo? and git_remote_diffs(branch)
end
