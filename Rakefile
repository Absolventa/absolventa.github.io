require 'active_support/core_ext'

posts_dir       = "_posts"    # directory for blog files
new_post_ext    = "md"  # default new post file extension when using the new_post task
author          = `finger $USER | sed -e '/Name/!d' -e 's/.*Name: //'` || "ABSOLVENTA Dev-Team"
# usage rake new_post[my-new-post] or rake new_post['my new post'] or rake new_post (defaults to "new-post")
desc "Begin a new post in #{posts_dir}"
task :new_post, :title do |t, args|
  if args.title
    title = args.title
  else
    title = get_stdin("Enter a title for your post: ")
  end
  title_url = title.parameterize

  filename = "#{posts_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{title_url}.#{new_post_ext}"
  if File.exist?(filename)
    abort("rake aborted!") if get_stdin("#{filename} already exists. Do you want to overwrite? ['y', 'n'] ") == 'n'
  end
  puts "Creating new post: #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: post"
    post.puts "title: \"#{title.gsub(/&/,'&amp;')}\""
    post.puts "date: #{Time.now.strftime('%Y-%m-%d %H:%M')}"
    post.puts "author: #{author}"
    post.puts "comments: true"
    post.puts "tags:"
    post.puts "  - ruby"
    post.puts "---"
  end
end


def get_stdin(message)
  print message
  STDIN.gets.chomp
end
