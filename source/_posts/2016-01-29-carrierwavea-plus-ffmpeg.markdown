---
layout: post
title: "用 carrierwave + FFMPEG 影片轉檔"
date: 2016-01-29 20:26:02 +0800
comments: true
categories: gem rails
---

若是上傳的檔案是影片，並且要對影片做其他處理，就可以使用 FFMPRG 來處理。

<!-- more --> 

#電腦先安裝ffmpeg

```
brew install ffmpeg
```

#Gem
```ruby
gem 'carrierwave'
gem 'streamio-ffmpeg'
```

#設定
`application.rb`

```ruby
require 'carrierwave'
```

`lib/carrierwave/ffmpeg.rb`

```ruby
require 'streamio-ffmpeg'
module CarrierWave
  module FFMPEG
    module ClassMethods
      def resample(bitrate)
        process :resample => bitrate
      end
    end

    def resample(bitrate)
      directory = File.dirname(current_path)
      tmpfile = File.join(directory, "tmpfile")

      FileUtils.mv( current_path, tmpfile )

      file = ::FFMPEG::Movie.new(tmpfile)
      file.transcode(current_path, video_bitrate: bitrate)

      File.delete(tmpfile)
    end
  end
end
```

`app/uploaders/video_uploader.rb`

```ruby
require File.join(Rails.root, "lib", "carrierwave", "ffmpeg")

class VideoUploader < CarrierWave::Uploader::Base
  include CarrierWave::FFMPEG

  def store_dir
    "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
  end

  def extension_white_list
    %w(mp4 flv)
  end

  version :bitrate_800k do
    process :resample => "800k"
  end

  version :bitrate_500k, from_version: :bitrate_800k do
    process :resample => "500k"
  end

end

```

#FFMPEG

也可以用 ruby 直接下 ffmpef 的指令

```ruby
ffmpeg -y -i #{input_path} -vf "scale=ceil(oh*a):480" -vcodec libx264 -preset:v slow -pix_fmt yuv420p -profile:v baseline -level 3.0 -b #{bitrate} -r 29.97 -acodec libvo_aacenc -ac 2 -ar 44100 -ab 64k -movflags faststart #{output_path}
```

縮圖

```ruby
ffmpeg -i #{input} -ss 00:00:05 -vframes 1 #{thumbnil}
```

#Calling shell commands from Ruby
1. Kernel#\`, commonly called backticks – `cmd` 

Returns the result of the shell command.

```ruby
value = `echo 'hi'`
value = `#{cmd}`
```

2. Built-in syntax, `%x( cmd )`  

Returns the result of the shell command, just like the backticks.

```ruby
value = %x( echo 'hi' )
value = %x[ #{cmd} ]
```
3. Kernel# `system`

Return: true if the command was found and ran successfully, false otherwise

```ruby
wasGood = system( "echo 'hi'" )
wasGood = system( cmd )
```
4. Kernel#exec

Return: none, the current process is replaced and never continues

```ruby
exec( "echo 'hi'" )
exec( cmd ) # Note: this will never be reached because of the line above
```


官方文件：  
[carrierwave](https://github.com/carrierwaveuploader/carrierwave)  
[streamio-ffmpeg](https://github.com/streamio/streamio-ffmpeg)  
[FFMPEG](https://www.ffmpeg.org/)  
[Kernel](http://ruby-doc.org/core-2.3.0/Kernel.html) 

參考文件：  
[Create FFMPEG processor for Carrierwave in Rails 3](http://www.freezzo.com/2010/12/23/create-ffmpeg-processor-for-carrierwave-in-rails-3/)  
[CarrierWave - basic video conversion](https://prograils.com/posts/carrierwave-basic-video-conversion)  
[Running command line commands within Ruby script](http://stackoverflow.com/questions/3159945/running-command-line-commands-within-ruby-script)  
[Calling shell commands from Ruby](http://stackoverflow.com/questions/2232/calling-shell-commands-from-ruby)  
[6 Ways to Run Shell Commands in Ruby Tuesday](http://tech.natemurray.com/2007/03/ruby-shell-commands.html)