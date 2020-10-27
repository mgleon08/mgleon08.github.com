---
layout: post
title: "Excel Report"
date: 2020-05-13 23:17:29 +0800
comments: true
categories: ruby
---

<!-- more -->

當需要產生報表時，可能就需要像是這種套件去產生，目前使用到的是 [spreadsheet](https://github.com/zdavatz/spreadsheet) 也有另一套星星數比較多的 [axlsx](https://github.com/randym/axlsx) 感覺比較多功能?

# Example

```ruby
require 'spreadsheet'
require 'pry'
require 'active_support/all'

module Reports
  class UsersImportFormGenerator
    WIDTH_RATIO = 1.freeze
    HEIGHT_RATIO = 3.freeze
    WIDTH_DEFAULT = 5.freeze
    DEFAULT_FORMAT = { size: 12, vertical_align: :top, horizontal_align: :left, family: :roman }.freeze

    # sheets params
    #
    # number_format: '@': string, '#': float, %: percentage
    # 如果沒有指定 number_format 預設 cell 都會是 auto, 但在 number 和 excel 行為不太一樣
    # number 一開始先給予超過 10 row，後面新增的 row 就會複製前面一個 row 的格式
    #
    # [{
    #   name: 'name'
    #   rows: [['col', 'col'], ['col', 'col']]
    #   options: {
    #       # specified_format > cell format > row format > col format > default_format
    #       default_format: { vertical_align: :top, horizontal_align: :left, size: 12 },
    #       rows_format: [{ idx: 0, format: { color: :yellow } }],
    #       columns_format: [{ idx: 0, format: { color: :red } }],
    #       specified_columns_format: [{ keys: ['Full Name'], format: { number_format: "@" } }],
    #       cells_format: [{ idx: 2, idy: 1, format: { color: :yellow } }],
    #       specified_format: [{ key: ['Name'], format: { color: :yellow } }],
    #       freeze: { top: 2, left: 0 },
    #       links: [{ idx: 1, idy: 1, url: 'www.google.com', text: 'google_link' }]
    #   }
    # }]

    def initialize(sheets = [])
      @sheets = sheets
      @spreadsheet = Spreadsheet::Workbook.new
      @spreadsheet.set_custom_color(21, 255, 199, 205)
      @spreadsheet.set_custom_color(34, 198, 240, 206)
    end

    def generate
      @sheets.each do |sheet|
        worksheet = @spreadsheet.create_worksheet(name: sheet[:name])
        options = sheet[:options]
        @default_format = (options && options[:default_format]) || DEFAULT_FORMAT
        set_freeze(worksheet, options[:freeze]) if options && options[:freeze]

        set_default_format(worksheet)
        insert_rows(sheet[:rows], worksheet)
        insert_links(worksheet, options[:links]) if options && options[:links]
        worksheet[3,0] = Spreadsheet::Link.new 'http://www.softwaremaniacs.net/2013/11/setting-cell-color-using-ruby.html', 'link text'
        basic_reformat(worksheet)
        custom_reformat(worksheet, sheet[:options])
      end
      @spreadsheet.write 'out.xlsx'
    end

    private

    def set_freeze(worksheet, freeze)
      worksheet.freeze!(freeze[:top], freeze[:left])
    end

    def set_default_format(worksheet)
      worksheet.protect!('123')
      worksheet.default_format = Spreadsheet::Format.new(@default_format)
    end

    def insert_rows(rows, worksheet)
      rows.each_with_index do |row, row_idx|
        worksheet.insert_row(row_idx, row)
      end
    end

    def insert_links(worksheet, links)
      links.each do |link|
        worksheet[link[:idy], link[:idx]] = Spreadsheet::Link.new(link[:url], link[:text])
      end
    end

    def basic_reformat(worksheet)
      adjust_height(worksheet)
      adjust_width(worksheet)
    end

    # cell format > row format > col format
    def custom_reformat(worksheet, options)
      custom_reformat_row(worksheet, options[:rows_format])             if options[:rows_format]
      custom_reformat_column(worksheet, (options[:columns_format] || []), (options[:specified_columns_format] || [])) if options[:columns_format] || options[:specified_columns_format]
      custom_reformat_cell(worksheet, options[:cells_format])           if options[:cells_format]
      custom_reformat_specified(worksheet, options[:specified_format])  if options[:specified_format]
    end

    def custom_reformat_row(worksheet, rows_format)
      rows_format.each do |row|
        worksheet.row(row[:idx]).default_format =
          Spreadsheet::Format.new(**@default_format, **row[:format])
      end
    end

    def custom_reformat_column(worksheet, columns_format, specified_columns_format)
      header = worksheet.row(0)
      format_columns = specified_columns_format.each_with_object(Hash.new { |h, k| h[k] = {} }) do |column, hash|
        map_column = column[:keys].map { |key| header.find_index(key) }
        map_column.each do |idx|
          hash[idx] = { **hash[idx], **column[:format] }
        end
      end

      columns_format.each do |column|
        format_columns[column[:idx]] = { **format_columns[column[:idx]], **column[:format] }
      end

      format_columns.each do |idx, format|
        worksheet.column(idx).default_format = Spreadsheet::Format.new({ **@default_format, **format })
      end
    end

    def custom_reformat_cell(worksheet, cells_format)
      formats = cells_format.map { |cell| cell[:format] }.uniq
      formats_mapping = formats.each_with_object({}) do |format, hash|
                          hash[format] = Spreadsheet::Format.new(**@default_format, **format)
                        end

      cells_format.each do |cell|
        worksheet.row(cell[:idy]).set_format(cell[:idx], formats_mapping[cell[:format]])
      end
    end


    def custom_reformat_specified(worksheet, specified_format)
      worksheet.each_with_index do |row, row_idx|
        row.each_with_index do |content, col_idx|
          specified_format.each do |specified|
            if specified[:key].include? content
              worksheet.row(row_idx).update_format(col_idx, specified[:format])
            end
          end
        end
      end
    end

    # basic_reformat
    def adjust_height(worksheet)
      [*0...worksheet.row_count].each do |row_idx|
        row = worksheet.row(row_idx)
        max_height = row.map { |cell| (cell.present? && cell.is_a?(String)) ? cell.split("\n").size : 1 }.max
        unless row.all? { |cell| cell.blank? }
          row.height =
            if max_height
              max_height > 200 ? 200 : row.height * max_height * HEIGHT_RATIO
            else
              row.height
            end
        end
      end
    end

    # basic_reformat
    def adjust_width(worksheet)
      [*0...worksheet.column_count].each do |col_idx|
        column = worksheet.column(col_idx)
        max_column = column.map { |cell| (cell.present? && cell.is_a?(String)) ? cell.split("\n").max_by(&:length).length : 1 }.max
        column.width =
          if max_column < 12
            12
          elsif max_column > 100
            100
          else
            max_column + WIDTH_DEFAULT
          end
      end
    end
  end
end
```

```ruby
ROW = [
  'Name',
  'Email',
  'Gender',
  'Birthday',
].freeze

ROW2 = [
  "Leon",
  "mgleon08@gmail.com",
  "Male",
  Date.new(2020, 1, 20)
].freeze

ROW3 = [
  "Hi",
  "",
  "Male",
  ""
].freeze


SHEET = [ROW, ROW2, ROW3]

8.times do
  SHEET << ["", "", "", ""].freeze
end

ALIGN = { vertical_align: :top, horizontal_align: :left }
DEFAULT_FORMAT = { **ALIGN, size: 13, color: :green , text_wrap: true }
ROWS_FORMAT = [{ idx: 0, format: { weight: :bold, size: 14, color: :orange, locked: true } }]
COLUMNS_FORMAT = [{ idx: 0, format: { color: :red } }]
SPECIFIED_COLUMNS_FORMAT =  [{ keys: ['Full Name'], format: { number_format: "@" } }]
CELLS_FORMAT = [
  { idx: 2, idy: 1, format: { color: :xls_color_9, pattern: 1, pattern_fg_color: :xls_color_34, border: :thin, border_color: :black } },
  { idx: 0, idy: 0, format: { color: :xls_color_8, pattern: 1, pattern_fg_color: :xls_color_21, border: :thin, border_color: :black } },
]
SPECIFIED_FORMAT =  [{ key: ['Full Name'], format: { color: :yellow } }]
FREEZE = { top: 2, left: 0 }
LINKS = [{ idx: 1, idy: 1, url: 'www.google.com', text: 'google_link' }]

sheets = [
  {
    name: 'UsersImportForm',
    rows: SHEET,
    options: {
      default_format: DEFAULT_FORMAT,
      rows_format: ROWS_FORMAT,
      columns_format: COLUMNS_FORMAT,
      specified_columns_format: SPECIFIED_COLUMNS_FORMAT,
      cells_format: CELLS_FORMAT,
      specified_format: SPECIFIED_FORMAT,
      freeze: FREEZE,
      links: LINKS
    }
  }
]

Reports::UsersImportFormGenerator.new(sheets).generate
```


```ruby
ruby "2.4.1"
source 'https://rubygems.org'

gem 'spreadsheet', '~> 1.1.4'
gem 'pry'
gem 'activesupport'
```

Reference: 

 * [spreadsheet](https://github.com/zdavatz/spreadsheet)
 * [axlsx](https://github.com/randym/axlsx)
 * [rubydoc](https://www.rubydoc.info/gems/ruby-spreadsheet/Spreadsheet)
 * [GUIDE](https://github.com/zdavatz/spreadsheet/blob/master/GUIDE.md)
 * [color](http://www.softwaremaniacs.net/2013/11/setting-cell-color-using-ruby.html )
 * [Direct downloading a xls file without writing it to the directory by Spreadsheet gem
](https://stackoverflow.com/questions/4049939/direct-downloading-a-xls-file-without-writing-it-to-the-directory-by-spreadsheet)
 * [Use custom colors with Spreadsheet gem
](https://stackoverflow.com/questions/33416887/use-custom-colors-with-spreadsheet-gem?rq=1)
* [color to hex](https://encycolorpedia.com/)
