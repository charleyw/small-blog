$:.unshift File.dirname(__FILE__) + '/sinatra/lib'

require 'rubygems'
require 'sinatra'
require 'RMagick'
require 'spec/rake/spectask'


desc "Run all specs"
Spec::Rake::SpecTask.new('specs') do |t|  
  t.spec_files = FileList['test/spec_*.rb']  
  t.spec_opts = ['--options', 'test/spec.opts']  
end

namespace 'server' do
  desc "Start production server on :5000"
  task :start do
    sh 'thin -s 1 -C config/config.yml -R rackup.ru start'
  end

  desc "Stop production server on :5000"
  task :stop do
    sh 'thin -s 1 -C config/config.yml -R rackup.ru stop'
  end
end

namespace 'db' do
  desc "Create db schema"
  task :create do        
    require 'activerecord'
    require 'config/config.rb'

    ENV.include?("mode") ? connect_to_db(ENV["mode"]) : connect_to_db(:development)
  
    ActiveRecord::Migration.create_table :posts do |t|
      t.string :title
      t.text :body
  
      t.timestamps
    end
  
    ActiveRecord::Migration.create_table :comments do |t|      
      t.text :body
      t.integer :post_id
  
      t.timestamps
    end

    ActiveRecord::Migration.create_table :tags do |t|    
      t.string :tag
  
      t.timestamps
    end

    ActiveRecord::Migration.create_table :posts_tags, :id => false do |t|
      t.integer "post_id", :null => false
      t.integer "tag_id",  :null => false
    end

  end
end

namespace 'gallery' do  
  GALLERY_IMAGES_DIR = "public/gallery"

  def create_thumbnail(image, gallery_name)    
    (image.columns >=  image.rows) ? crop_factor = image.rows : crop_factor = image.columns    

    preview = image.crop_resized(crop_factor, crop_factor, Magick::NorthGravity)
    preview.thumbnail!(100, 100)
    
    filename = File.basename(image.base_filename)
    FileUtils.mkdir_p File.join(Dir.pwd, "#{GALLERY_IMAGES_DIR}/#{gallery_name}/previews")
    preview.write("#{GALLERY_IMAGES_DIR}/#{gallery_name}/previews/#{filename}") { self.quality = 95 }    
  end

  def create_resized_image(image, gallery_name)
    photo = image.resize_to_fit(800, 800)
    filename = File.basename(image.base_filename)
    FileUtils.mkdir_p File.join(Dir.pwd, "#{GALLERY_IMAGES_DIR}/#{gallery_name}")
    photo.write("#{GALLERY_IMAGES_DIR}/#{gallery_name}/#{filename}") { self.quality = 95 }
  end

  def process_directory(dir, gallery_name)    
    Dir.foreach(dir) do |image|    
      if image.downcase.include? ".jpg"
        puts "Processing #{dir}/#{image}" 
        
        image = Magick::Image.read("#{dir}/#{image}").first        
        create_thumbnail(image, gallery_name)
        create_resized_image(image, gallery_name)        
      end
    end
  end

  desc "Fill gallery with images."
  task 'fill' do
    unless ENV.include?("src")
      raise "usage rake gallery:fill src= #folder with images"
    end

    FileUtils.rm_rf GALLERY_IMAGES_DIR
    FileUtils.mkdir_p GALLERY_IMAGES_DIR

    src_dir = ENV["src"]    

    Dir.foreach(src_dir) do |gallery_dir| 
      gallery_path = File.join(File.join(Dir.pwd, src_dir), gallery_dir) 

      if (File.directory?(gallery_path))        
        process_directory(gallery_path, gallery_dir)  
      end
    end
  end
end
