require 'rubygems'
require 'bundler/setup'

require 'rake/clean'

distro = 'rpm'
fpm_opts = ""

version = "v2010.02.14"
release = Time.now.utc.strftime('%Y%m%d%H%M%S')
name = "autoconf-archive-#{version}"

description_string = %Q{The GNU Autoconf Archive is a collection of more than 450 macros for GNU
Autoconf < http://www.gnu.org/software/autoconf> _ that have been contributed as
free software by friendly supporters of the cause from all over the Internet.}

jailed_root = File.expand_path('../jailed-root', __FILE__)

CLEAN.include("downloads")
CLEAN.include("jailed-root")
CLEAN.include("log")
CLEAN.include("pkg")

task :init do
  mkdir_p "log"
  mkdir_p "pkg"
  mkdir_p "downloads"
  mkdir_p "jailed-root"
end

task :download do
  cd 'downloads' do
    sh("git clone --quiet https://github.com/peti/autoconf-archive.git autoconf-archive-#{version}")
    cd "autoconf-archive-#{version}" do
      sh("git checkout tags/#{version}")
      sh("git clone --quiet --depth=1 git://git.savannah.gnu.org/gnulib.git")
    end
  end
end

task :dependencies do
  sh("sudo yum install -y coreutils python3 texlive texinfo tidy")
end

task :configure do
  cd "downloads" do
    cd "autoconf-archive-#{version}" do
      sh("./bootstrap.sh")
      sh "./configure --prefix=/usr > #{File.dirname(__FILE__)}/log/configure.#{version}.log 2>&1"
    end
  end
end

task :make do
  num_processors = %x[nproc].chomp.to_i
  num_jobs       = num_processors + 1

  cd "downloads/autoconf-archive-#{version}" do
    sh("make -j#{num_jobs} maintainer-generate > #{File.dirname(__FILE__)}/log/make.#{version}.log 2>&1")
    sh("make -j#{num_jobs} > #{File.dirname(__FILE__)}/log/make.#{version}.log 2>&1")
  end
end

task :make_install do
  rm_rf  jailed_root
  mkdir_p jailed_root
  cd "downloads/autoconf-archive-#{version}" do
    sh("sudo make install DESTDIR=#{jailed_root} > #{File.dirname(__FILE__)}/log/make-install.#{version}.log 2>&1")
  end
end


task :dist do
  require 'erb'
  class RpmSpec
    attr_accessor :version, :release
    def initialize(version, release)
      @version      = version
      @release      = release
    end

    def get_binding
      binding
    end
  end
  sh("rm -rf #{jailed_root}/usr/share/info/dir")

  cd "pkg" do
    sh(%Q{
      bundle exec fpm -s dir -t #{distro} --name autoconf-archive-#{version} -a x86_64 --version "#{version}" -C #{jailed_root} --verbose #{fpm_opts} --maintainer snap-ci@thoughtworks.com --vendor snap-ci@thoughtworks.com --url http://snap-ci.com --description "#{description_string}" --iteration #{release} --license 'GPLv2' .
    })
  end
end

desc "build autoconf-archive rpm/deb"
task :default => [:clean, :init, :download, :dependencies, :configure, :make, :make_install, :dist]
