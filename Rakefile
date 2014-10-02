namespace :post do

  desc 'create a new post'
  task :new do

    print 'title: '
    title = STDIN.gets.strip
    date = Time.now.strftime('%Y-%m-%d')

    metadata = { title: title,
      date: date
    }

    filename = File.join('_posts', "#{date}-#{title.tr(' ', '-')}.md")

    fail "#{filename} already exists" if File.exist? filename

    File.open(filename, 'w') do |handle|
      handle.puts "---"
      handle.puts 'title: title'
      handle.puts 'layout: post'
      handle.puts "---\n\n\n"
    end

    system "$EDITOR +6 #{filename}"

  end

end
