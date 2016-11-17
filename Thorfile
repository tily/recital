require "cgi"
require "uri"
require "thor"
require "nokogiri"
require "open-uri"
require "toml"

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
      t = "%02d:%02d.%s" % [start / 60, start % 60, (start - start.to_i).round(2).to_s.gsub(/^0\./, "")]
      %Q|{{<t "#{t}">}}|
    end
    File.write(path, content)
  end

  desc "lookup", "get a word definition from alc"
  def lookup(word)
    doc = Nokogiri::HTML(open("http://eow.alc.co.jp/#{CGI.escape(word)}").read)
    list = doc.search("#resultsList ul li")
    list.reverse.each do |item|
      title = item.search("h2").text
      next if title == ""
      definitions = item.search("ol li").map{|li| li.text.gsub(/\n+/, "\n") }
      definitions += item.search("div").map{|li| li.text }
      print "#{title} = "
      puts definitions.map{|d| "#{d}" }.join(" / ")
    end
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

  desc "separate", "separate words"
  def separate(path)
    rewrite_file(path) do |content|
      buffer = []
      content.split("\n").each do |line|
        if match = line.match(/^(\* \{\{<t .+>\}\} )(.+)/)
          prefix, content = match[1], match[2]
          content = content.gsub(/([\w']+)/) { "{{<w \"#{$1}\">}}" }
          buffer << prefix + content
        else
          buffer << line
        end
      end
      next buffer.join("\n")
    end
  end

  desc "pronounce", "get pronunciations of words and save to config.toml"
  def pronounce(path)
    rewrite_toml("config.toml") do |toml|
      pronunciation = toml["params"]["pronunciation"]

      File.open(path).each do |line|
        words = line.scan(/\{\{<w "(.+?)">\}\}/).map{|a| a.first.downcase }
        words.each do |word|
          begin
            if !pronunciation[%Q("#{word}")]
              pron = get_pronunciation(word)
              puts %Q("#{word}" = "#{pron}")
              pronunciation[%Q("#{word}")] = pron
            end
          rescue => e
            pronunciation[%Q("#{word}")] = ""
            puts e
          end
        end
      end
      next toml
    end
  end

  no_commands do
    def get_pronunciation(word)
      doc = Nokogiri::HTML(open("http://ejje.weblio.jp/content/#{CGI.escape(word)}").read)
      res = doc.search("span.phoneticEjjeDesc").first
      res.search("span.phoneticEjjeExt").remove
      return res.text.split(/;/).first
    end

    def rewrite_file(path)
      original = File.read(path)
      rewritten = yield(original)
      File.write(path, rewritten)
    end

    def rewrite_toml(path)
      rewrite_file(path) do |original|
        original = TOML.load(original)
        rewritten = yield(original)
        next TOML::Generator.new(rewritten).body
      end
    end
  end
end
