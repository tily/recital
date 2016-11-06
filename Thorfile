require "cgi"
require "uri"
require "thor"
require "nokogiri"
require "open-uri"

class Default < Thor
  desc "download", "download genius lyrics"
  def download(url, path)
    doc = Nokogiri::HTML(open(url).read)
    doc.search("script").remove
    title = doc.search("h2").text.gsub(/\n/, "").gsub(/\s+/, " ")
    lyrics = doc.search("lyrics").text.gsub(/\n/, "  \n")
    meta = <<-EOS
+++
draft = false
date = "#{Time.now.iso8601}"
title = "#{title}"
+++
    EOS
    File.write(path, meta + lyrics)
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
