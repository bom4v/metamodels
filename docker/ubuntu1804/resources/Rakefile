require 'yaml'
require 'nokogiri'

# MetaModels root path
ROOT_PATH = File.expand_path File.dirname(__FILE__)

# Load the YAML configuration file
CONF_FILE = File.join ROOT_PATH, 'metamodels.yaml'
CONF = File.open(CONF_FILE) { |f| YAML.load f }
raise "Unable to load YAML configuration file from #{CONF_FILE}" unless CONF
CONF.each do |param, val|
  Object.const_set "CONF_#{param.upcase}", val
end

[
  'components',
  'default_branch',
  'default_base_repo',
  'default_buildbin',
  'mvn_args',
  'sbt_args'
].each do |param|
  raise "Missing mandatory parameter '#{param}' in configuration file." unless CONF[param]
end

COMPONENTS = CONF_COMPONENTS.map { |c| c['name'] }

# MetaModels workspace
WORK_PATH = File.join ROOT_PATH, 'workspace'

# Required binaries
BINARIES = ['git', 'mvn', 'sbt']
BINARIES.each do |bin|
  bin_path = %x[ which #{bin} ].strip
  raise "Executable #{bin} not found!" unless File.executable?(bin_path)
  Object.const_set "#{bin.upcase}_BIN", bin_path
end

# Path to cloned sources
def component_src_path(cmp)
  File.join WORK_PATH, 'src', cmp
end

# Lookup the builder binary of a component (either Maven or SBT)
def component_build_bin(cmp)
  component(cmp).fetch('buildbin', CONF_DEFAULT_BUILDBIN)
end

# Path to build output
def component_build_path(cmp)
  File.join WORK_PATH, 'build', cmp
end

# Run a shell command
def do_shell(cmd)
  puts "[METAMODELS]:#{Dir.pwd}$ #{cmd}"
  raise "Shell command failure" unless system(cmd)
end

def component(cmp)
  CONF_COMPONENTS.select { |c| c['name'] == cmp }.first
end

# Lookup the Git branch of a component
def component_branch(cmp)
  component(cmp).fetch('branch', CONF_DEFAULT_BRANCH)
end

# Lookup the build dependencies of a component
def component_deps(cmp)
  component(cmp).fetch('deps', [])
end

# Lookup the repository of a component
def component_repo(cmp)
  component(cmp).fetch('repo', "#{CONF_DEFAULT_BASE_REPO}/#{cmp}.git")
end

# Lookup the targetted Hadoop versions of a component
def component_hadoop_versions(cmp)
  component(cmp).fetch('hadoop_versions', CONF_DEFAULT_HADOOP_VERSIONS)
end

# Lookup the targetted Spark versions of a component
def component_spark_versions(cmp)
  component(cmp).fetch('spark_versions', CONF_DEFAULT_SPARK_VERSIONS)
end

# Lookup Maven POM file
def lookup_maven_pom(cmp_src_path)
  mvn_hadoop_versions = Array.new
  mvn_versions = Array.new
  if File.executable?(cmp_src_path)
    ::Dir.chdir(cmp_src_path) do
      CONF_DEFAULT_HADOOP_VERSIONS.each do |hadoop_version|
        mvn_filename = 'pom.hadoop' + hadoop_version.to_s + '.xml'
        if File.readable?(mvn_filename)
          mvn_hadoop_versions << hadoop_version.to_s
          mvn_versions << hadoop_version.to_s
        else
        end
      end
      if File.readable?('pom.xml')
        if mvn_hadoop_versions.empty?
          mvn_versions << ''
        end
      end
    end
  end
  mvn_versions
end

# Lookup SBT file
def lookup_sbt_file(cmp_src_path)
  sbt_file = ''
  if File.executable?(cmp_src_path)
    ::Dir.chdir(cmp_src_path) do
      if File.readable?('build.sbt')
        sbt_file = File.join cmp_src_path, 'build.sbt'
      end
    end
  end
  sbt_file
end

# Extract the version from a Maven POM file
def lookup_version_from_mvn_pom(mvn_pom_file)
  mvn_pom_version = '0.0.0'
  has_hadoop = false
  has_spark = false
  if File.readable?(mvn_pom_file)
    doc = File.open(mvn_pom_file) { |f| Nokogiri::XML(f) }
    mvn_pom_full_version = doc.at_css('//version').content
    if mvn_pom_full_version =~ /^(\d*\.\d*.\d*)(-hadoop\d\.\d)?(-spark\d\.\d.\d)?$/i
      mvn_pom_version = $1
      if $2
        has_hadoop = true
      end
      if $3
        has_spark = true
      end
    end
  end
  [mvn_pom_version, has_hadoop, has_spark]
end

# Extract the version from running a dedicated SBT target
def lookup_version_from_sbt_build(sbt_build_file)
  sbt_proj_version = '0.0.0'
  has_hadoop = false
  has_spark = false
  File.open(sbt_build_file, "r") do |f|
    file_content = f.read
    if file_content =~ /version := "(\d*\.\d*.\d*)(-hadoop\d\.\d)?(-spark\d\.\d.\d)?"[,]?$/i
      sbt_proj_version = $1
      if $2
        has_hadoop = true
      end
      if $3
        has_spark = true
      end
    end
  end
  [sbt_proj_version, has_hadoop, has_spark]
end

# Set up the Maven build targets
def setup_maven_build_targets(cmp, build_vars)
  cmp_src_path = component_src_path cmp
  cmp_build_bin = component_build_bin cmp

  if cmp_build_bin != 'mvn'
    STDERR.puts "[#{cmp}]: the builder is configured (in the YAML file) as '#{cmp_build_bin}', however it seems to be 'mvn'. Look in the source directory ('#{cmp_src_path}') for more details"
  end

  mvn_versions = lookup_maven_pom(cmp_src_path)
  mvn_versions.each do |mvn_version|
    mvn_filename = 'pom.xml'
    if mvn_version != ''
      mvn_filename = 'pom.hadoop' + mvn_version + '.xml'
    end
    mvn_file = File.join cmp_src_path, mvn_filename

    # Store the Maven POM file
    build_vars['BUILDER_FILES'] << mvn_file

    # Extract the elements of version from the Maven POM XML file
    (mvn_version, has_hadoop, has_spark) = lookup_version_from_mvn_pom(mvn_filename)

    # Store the elements of version
    build_vars['PACKAGE_VERSIONS'][mvn_version] = true
    build_vars['HAS_HADOOP'] = has_hadoop
    build_vars['HAS_SPARK'] = has_spark

    # Set up the Maven build command line
    cmp_builder_cmds = "#{cmp_build_bin} -f #{mvn_file} #{CONF_MVN_ARGS}"
    build_vars['BUILDER_CMDS'] << "#{cmp_builder_cmds}"
    
    # Maven tasks/targets for building, packaing and installing (in the local
    # Maven repository, eg, ~/.m2/repository)
    cmp_builder_tasks = "compile package install"
    build_vars['BUILDER_FULL_CMDS'] << "#{cmp_builder_cmds} #{cmp_builder_tasks}"

    # Set up the Maven test command line
    cmp_test_tasks = "test"
    build_vars['TEST_FULL_CMDS'] << "#{cmp_builder_cmds} #{cmp_test_tasks}"
  end
end

# Set up the SBT build targets
def setup_sbt_build_targets(cmp, build_vars)
  cmp_src_path = component_src_path cmp
  cmp_build_bin = component_build_bin cmp
  cmp_hadoop_versions = component_hadoop_versions cmp
  cmp_spark_versions = component_spark_versions cmp

  # SBT build (for Scala code)
  sbt_file = lookup_sbt_file(cmp_src_path)
  if sbt_file != ''
    if cmp_build_bin != 'sbt'
      STDERR.puts "[#{cmp}]: the builder is configured (in the YAML file) as '#{cmp_build_bin}', however it seems to be 'sbt'. Look in the source directory ('#{cmp_src_path}') for more details"
    end

    # Store the SBT build file
    build_vars['BUILDER_FILES'] << sbt_file

    # Extract the elements of version from the SBT build file
    (sbt_version, has_hadoop, has_spark) = lookup_version_from_sbt_build(sbt_file)

    # Store the elements of version
    build_vars['PACKAGE_VERSIONS'][sbt_version] = true
    build_vars['HAS_HADOOP'] = has_hadoop
    build_vars['HAS_SPARK'] = has_spark

    # Set up the SBT build command line
    sbt_args_hadoop_list = Array.new
    sbt_args_list = Array.new
    sbt_args_base = "\'; #{CONF_SBT_ARGS} ; set version := \"#{sbt_version}"
    if has_hadoop
      # There should be a target for every version of Hadoop
      cmp_hadoop_versions.each do |hadoop_version|
        sbt_args = "#{sbt_args_base}-hadoop#{hadoop_version}"
        sbt_args_hadoop_list << sbt_args
      end
    end

    if has_spark
      # When building for Spark, Hadoop is necessarily part of the target
      cmp_spark_versions.each do |spark_version|
        sbt_args_hadoop_list.each do |sbt_args|
          sbt_args += "-spark#{spark_version}"
          sbt_args_list << sbt_args
        end
      end

    else
      # Target for Hadoop, but not Spark
      sbt_args_hadoop_list.each do |sbt_args|
        sbt_args_list << sbt_args
      end
    end

    if sbt_args_list.empty?
      # Target with Hadoop, and potentially Spark
      sbt_args = sbt_args_base + "\"\'"

      cmp_builder_cmds = "#{cmp_build_bin} #{sbt_args}"
      build_vars['BUILDER_CMDS'] = "#{cmp_builder_cmds}"

      cmp_builder_tasks = "+compile +package +publish-m2 +publish-local"
      build_vars['BUILDER_FULL_CMDS'] << "#{cmp_builder_cmds} #{cmp_builder_tasks}"
      # Set up the SBT test command line
      cmp_test_tasks = "+test"
      build_vars['TEST_FULL_CMDS'] << "#{cmp_builder_cmds} #{cmp_test_tasks}"

    else
      # Target with neither Hadoop nor Spark
      sbt_args_list.each do |sbt_args|
        sbt_args += "\"\'"

        cmp_builder_cmds = "#{cmp_build_bin} #{sbt_args}"
        build_vars['BUILDER_CMDS'] = "#{cmp_builder_cmds}"
        
        cmp_builder_tasks = "+compile +package +publish-m2 +publish-local"
        build_vars['BUILDER_FULL_CMDS'] << "#{cmp_builder_cmds} #{cmp_builder_tasks}"
        
        # Set up the SBT test command line
        cmp_test_tasks = "+test"
        build_vars['TEST_FULL_CMDS'] << "#{cmp_builder_cmds} #{cmp_test_tasks}"
      end
    end

  else
    STDERR.puts "[#{cmp}]: cannot determine the builder in '#{cmp_src_path}'"
  end
end

# Lookup the component versions
def lookup_build_vars(cmp)
  cmp_src_path = component_src_path cmp
  cmp_build_path = component_build_path cmp
  cmp_build_bin = component_build_bin cmp
  build_vars = {}
  build_vars['BUILDER_BIN'] = cmp_build_bin
  build_vars['BUILDER_FILES'] = Array.new
  build_vars['PACKAGE_VERSIONS'] = {}
  build_vars['HAS_HADOOP'] = false
  build_vars['HAS_SPARK'] = false
  build_vars['BUILDER_CMDS'] = Array.new
  build_vars['BUILDER_FULL_CMDS'] = Array.new
  build_vars['TEST_FULL_CMDS'] = Array.new

  if File.executable?(cmp_src_path)
    ::Dir.chdir(cmp_src_path) do
      mvn_versions = lookup_maven_pom(cmp_src_path)
      if not mvn_versions.empty?
        # Maven build (for Java code)
        setup_maven_build_targets(cmp, build_vars)

      else
        # SBT build (for Scala code)
        setup_sbt_build_targets(cmp, build_vars)
      end
    end
  end
  build_vars
end  
  
COMPONENTS_BUILD_VARS = {}

#
def component_build_vars(cmp)
  COMPONENTS_BUILD_VARS[cmp] ||= lookup_build_vars cmp
end

# Versions of a component
def component_versions(cmp)
  component_build_vars(cmp)['PACKAGE_VERSIONS']
end

# Whether or not the build of a component should be built against Hadoop
def component_has_hadoop(cmp)
  component_build_vars(cmp)['HAS_HADOOP']
end

# Whether or not the build of a component should be built against Spark
def component_has_spark(cmp)
  component_build_vars(cmp)['HAS_SPARK']
end

# Builder files of a component
def component_builder_files(cmp)
  component_build_vars(cmp)['BUILDER_FILES']
end

# Builder binary of a component
def component_builder_bin(cmp)
  component_build_vars(cmp)['BUILDER_BIN']
end

# Builder command of a component
def component_builder_commands(cmp)
  component_build_vars(cmp)['BUILDER_CMDS']
end

# Builder full commands of a component
def component_builder_full_commands(cmp)
  component_build_vars(cmp)['BUILDER_FULL_CMDS']
end

# Test full commands of a component
def component_test_full_commands(cmp)
  component_build_vars(cmp)['TEST_FULL_CMDS']
end


# Status task

desc "Display configuration information"
task :info do
  puts "Components:"
  COMPONENTS.each do |cmp|
    cmp_versions = component_versions cmp
    cmp_has_hadoop = component_has_hadoop cmp
    cmp_has_spark = component_has_spark cmp
    cmp_repo = component_repo cmp
    cmp_branch = component_branch cmp
    cmp_deps = component_deps cmp
    cmp_src_path = component_src_path cmp
    cmp_build_bin = component_build_bin cmp
    cmp_build_files = component_builder_files cmp
    cmp_hadoop_versions = component_hadoop_versions cmp
    cmp_spark_versions = component_spark_versions cmp
    puts " * #{cmp}"
    puts "   Release version        : #{cmp_versions.keys.first}" unless cmp_versions.empty?
    puts "   Git repository         : #{cmp_repo}"
    puts "   Checkout branch        : #{cmp_branch}"
    puts "   Depends on             : #{cmp_deps.join ', '}" unless cmp_deps.empty?
    puts "   Source cloned at       : #{cmp_src_path}"
    puts "   Built with             : #{cmp_build_bin}"
    puts "   Built files            : #{cmp_build_files.join ', '}" unless cmp_build_files.empty?
    puts "   Built for Hadoop       : #{cmp_has_hadoop}"
    puts "   Built for Spark        : #{cmp_has_spark}"
    puts "   Target Hadoop versions : #{cmp_hadoop_versions.join ', '}" unless cmp_hadoop_versions.empty?
    puts "   Target Spark versions  : #{cmp_spark_versions.join ', '}" unless cmp_spark_versions.empty?
    puts
  end
end

directory WORK_PATH

# Define all global tasks

SIMPLE_GIT_TASKS = [
  :status, :push, :pull
]
COMPLEX_TASKS = [
  :clone, :checkout, :clean, :deliver, :test
]
(SIMPLE_GIT_TASKS + COMPLEX_TASKS).each do |t|
  desc "#{t.to_s.capitalize} on all components"
  task t
end

# Define specific tasks for every component

COMPONENTS.each do |cmp|
  cmp_versions = component_versions cmp
  cmp_repo = component_repo cmp
  cmp_branch = component_branch cmp
  cmp_deps = component_deps cmp
  cmp_src_path = component_src_path cmp
  cmp_build_path = component_build_path cmp
  cmp_build_bin = component_build_bin cmp
  cmp_builder_cmds = component_builder_commands cmp
  cmp_builder_full_cmd_list = component_builder_full_commands cmp
  cmp_test_full_cmd_list = component_test_full_commands cmp
  cmp_hadoop_versions = component_hadoop_versions cmp
  cmp_spark_versions = component_spark_versions cmp
  
   
  # Clone tasks
  
  file cmp_src_path => WORK_PATH do
    do_shell "#{GIT_BIN} clone -n #{cmp_repo} #{cmp_src_path}"
  end
  desc "Clone component #{cmp} in #{cmp_src_path}"
  task "clone_#{cmp}" => cmp_src_path
  task :clone => "clone_#{cmp}"

  # Wrappers for simple git tasks
    
  SIMPLE_GIT_TASKS.each do |gittask|
    cmp_task = "#{gittask}_#{cmp}"
    desc "#{gittask.to_s.capitalize} on component #{cmp} in #{cmp_src_path}"
    task cmp_task => cmp_src_path do
      ::Dir.chdir(cmp_src_path) do
        do_shell "#{GIT_BIN} #{gittask}"
      end
    end
    task gittask => cmp_task
  end
  
  # Checkout tasks
  
  desc "Checkout branch #{cmp_branch} of component #{cmp}"
  task "checkout_#{cmp}" do
    ::Dir.chdir(cmp_src_path) do
      do_shell "#{GIT_BIN} checkout #{cmp_branch}"
    end
  end
  task("checkout_#{cmp}" => "pull_#{cmp}") unless ENV['offline']
  task :checkout => "checkout_#{cmp}"
  
  # Clean tasks
  
  desc "Clean the build environment of #{cmp}"
  task "clean_#{cmp}" do
    ::Dir.chdir(cmp_src_path) do
      do_shell "#{cmp_builder_cmds.first} clean"
    end
  end
  task :clean => "clean_#{cmp}"
  
  # Compilation tasks

  compilation_prereqs = ["checkout_#{cmp}"] + cmp_deps.map { |dep| "deliver_#{dep}" }
  
  task "deliver_#{cmp}" => compilation_prereqs do
    mkdir_p cmp_build_path
    ::Dir.chdir(cmp_src_path) do
      cmp_builder_full_cmd_list.each do |cmp_builder_full_cmd|
        do_shell "#{cmp_builder_full_cmd}"
      end
    end
  end
  desc "Deliver component #{cmp} in #{cmp_build_path}"
  task :deliver => "deliver_#{cmp}"

  # Test tasks

  task "test_#{cmp}" => compilation_prereqs do
    mkdir_p cmp_build_path
    ::Dir.chdir(cmp_src_path) do
      cmp_test_full_cmd_list.each do |cmp_test_full_cmd|
        do_shell "#{cmp_test_full_cmd}"
      end
    end
  end
  desc "Test component #{cmp} in #{cmp_build_path}"
  task :test => "test_#{cmp}"
 
end

task :default => :deliver
