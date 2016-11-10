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
p embed
    if content.match(/{{<y (.+?)>}}/)
      content = content.gsub(/{{<y (.+?)>}}/) { embed }
    else
      content = [content, embed].join("\n") 
    end
  end

  desc "annotate", "annotate timings to video"
  def annotate(bpm, start, path)
    spb = 60.0 / bpm.to_f
    start = start.to_f - spb * 4
    content = File.read(path)
    content.gsub!(/{{<t (.+?)>}}/) do
      start += spb * 4
      %Q|{{<t "#{start.round(2)}">}}|
    end
    File.write(path, content)
  end

  desc "convert", "convert"
  def convert(path)
    content = File.read(path)
    content.gsub!(/{{<t "(.+)">}}/) {
      orig = $1
p orig
      ms = orig.to_f.to_s[/\.(\d+)/, 1]
      sec = orig.to_i % 60
      min = orig.to_i / 60
      %q({{<t "%02d:%02d.%s">}}) % [min, sec, ms]
    }
    File.write(path, content)
  end
end
