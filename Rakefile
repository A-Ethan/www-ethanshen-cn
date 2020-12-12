# # # # # # # # # # # # # # # # # # # # #
#                                       #
#     windows 太难了，等有机会再用吧      #
#                                       #
#   传值大改不想改了，ruby win运行太难了   #
#                                       #
# # # # # # # # # # # # # # # # # # # # #


# Add 2020-12-12 
desc "Create a post in _drafts"
task :new do
    puts "Input file(for Url)："
    @url = STDIN.gets.chomp
    # puts "Input Title(for content)："
    # @name = STDIN.gets.chomp
    # puts "Input subtitle(for content)："
    # @subtitle = STDIN.gets.chomp
    # puts "Input Tags(for content | Separated By ,)"
    # @tags = STDIN.gets.chomp
    # puts "Input Keywords(for content | Separated By ,)"
    # @keywords = STDIN.gets.chomp

    @slug = "#{@url}"
    @slug = @slug.downcase.strip.gsub(' ', '-')
    @date = Time.now.strftime("%F")
    @post_name = "_drafts/#{@date}-#{@slug}.md"

# if File.exist?(@post_name)
#    abort("Failed to create the file name already exists !")
# end
# FileUtils.touch(@post_name)
# open(@post_name, 'a') do |file|
#     file.puts "---"
#     file.puts "title: #{@name}"
#     file.puts "layout: post"
#     file.puts "author: \"Ethan\""
#     file.puts "subtitle: #{@subtitle}"
#     file.puts "date: #{Time.now}"
#     file.puts "catalog: true"
#     file.puts "header-img: \"images/about-bg.jpg\""
#     file.puts "tags: #{@tags}"
#     file.puts "keywords: #{@keywords}"
#     file.puts "---"
# end
# windows type nul >  linux vi
exec "type nul > #{@post_name}" 
end