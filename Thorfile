require "cgi"
require "uri"
require "thor"
require "nokogiri"
require "open-uri"

class Default < Thor
  desc "download", "download lyrics"
  def download(url)
    doc = Nokogiri::HTML(open(url).read)
    doc.search("script").remove
    title = doc.search("h2").text.gsub(/\n/, "").gsub(/\s+/, " ")
    lyrics = doc.search("lyrics").text.gsub(/\n/, "  \n")
    puts <<-EOS
+++
draft = false
date = "#{Time.now.iso8601}"
title = "#{title}"
+++
    EOS
    puts lyrics
  end

  desc "embed", "embed youtube video to page"
  def embed(url, path)
    video_id = CGI.parse(URI.parse(url).query)["v"].first
    content = File.read(path)
    embed = "{{<y #{video_id}>}}"
    if content.match(/{{<y (.+?)>}}/)
      content = content.gsub(/{{<y (.+?)>}}/) { embed }
    else
      content = [content, embed].join("\n") 
    end
    File.write(path, content)
  end
end
