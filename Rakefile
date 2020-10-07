require 'bundler'
Bundler.setup

require 'rake'
require 'fileutils'
require 'openstudio'
require 'parallel'

#task default: 'tbd'

def clear_run
  puts 'Deleting run diretory and underlying contents'

  # remove run directory
  FileUtils.rm_rf('run')
end

desc 'Delete contents under run directory'
task :clear_run do
  clear_run
end

def find_bundle_measure_paths
  bundle_base_gem_path = '.bundle/install/ruby/2.5.0/bundler/gems'
  bundle_measure_paths = []

  puts "Getting measure directories for bundle installed measure gems"
  gems = Dir.entries(bundle_base_gem_path)
  gems.each do |gem|
    # check if has lib/measures
    gem = "#{bundle_base_gem_path}/#{gem}/lib/measures"
    next if ! Dir.exists?(gem)
    bundle_measure_paths << gem
  end

  puts "found #{bundle_measure_paths.size} measure directories"
  puts "bundle_measure_paths:#{bundle_measure_paths.inspect}"

  return bundle_measure_paths
end

desc 'Find Bundle measure paths to add to bundle osws'
task :find_bundle_measure_paths do
  find_bundle_measure_paths
end

def setup_osw(workflow_name)
  puts "Adding copy in run/workflows directory of #{workflow_name} in workflow directory with updated measure paths set to use .bundle measure gems."

  # make directory if does not exist
  FileUtils.mkdir_p("run/workflows/#{workflow_name}")

  # load OSW file
  osw = OpenStudio::WorkflowJSON.load("workflows/#{workflow_name}/in.osw").get
  runner = OpenStudio::Measure::OSRunner.new(osw)
  workflow = runner.workflow

  # replace measure paths, add in measure gem paths and measures from this repo
  puts "updating measure_paths to use the bundle measure gems"
  workflow.resetMeasurePaths
  find_bundle_measure_paths.each do |path|
    workflow.addMeasurePath("../../../#{path}")
  end

  # path to measures in this repo
  workflow.addMeasurePath("../../../measures")

  # update other paths in the osw for new location (file_paths should be one level deeper)
  puts "updating file_paths to adjust for location of copied osw file."
  workflow.resetFilePaths
  # storing workflow.filePaths and then looping through them creates lots of extras, hard coded for now
  workflow.addFilePath("../../../weather")
  workflow.addFilePath("../../../seeds")
  workflow.addFilePath("../../../files")

  # generally should not need to use paths in measure arguments if use findFile within the measure

  # save workflow
  puts "saving modified workflow"
  osw_path = "run/workflows/#{workflow_name}/in.osw"
  workflow.saveAs(osw_path)

  return workflow
end

desc 'Setup single osw file to use bundler gems for measure paths'
task :setup_osw , [:workflow_name] do |task, args|
  args.with_defaults(workflow_name: 'bar_typical')
  workflow_name = args.workflow_name.inspect.delete('"')
  setup_osw(workflow_name)
end

desc 'Setup all osw files to use bundler gems for measure paths'
task :setup_all_osws do
  find_osws.each do |workflow_name|
    setup_osw(workflow_name)
  end
end

def find_osws
  puts "Get names of workflows in workflows directory"
  workflow_names = []
  workflows = Dir.entries('workflows')
  workflows.each do |workflow|
    # check if has lib/measures
    workflow_path = "workflows/#{workflow}/in.osw"
    next if ! File.exists?(workflow_path)
    workflow_names << workflow
  end
  puts workflow_names

  return workflow_names
end

desc 'List OSW files in the measures workflows directory'
task :find_osws do
  find_osws
end

# todo - can get rid of this and always use run_osws unless people want to use without parallel
def run_osw(workflow_name, measures_only = false)
  puts "Running #{workflow_name}"
  if measures_only
    system("openstudio run -m -w run/workflows/#{workflow_name}/in.osw")
  else
    system("openstudio run -w run/workflows/#{workflow_name}/in.osw")
  end
end

def run_osws(workflow_names, measures_only = false)
  jobs = []
  workflow_names.each do |workflow_name|
    puts "Running #{workflow_name}"
    if measures_only
      jobs << "openstudio run -m -w run/workflows/#{workflow_name}/in.osw"
    else
      jobs << "openstudio run -w run/workflows/#{workflow_name}/in.osw"
    end
  end

  # run simulations
  num_parallel = 12 # can it default or input something like n-1
  Parallel.each(jobs, in_threads: num_parallel) do |job|
    puts job
    system(job)
  end

end

desc 'Run single osw'
task :run_osw , [:workflow_name] do |task, args|
  args.with_defaults(workflow_name: 'bar_typical')
  workflow_name = args.workflow_name.inspect.delete('"')
  puts "Running #{workflow_name}"
  run_osw(workflow_name)
end

desc 'Run all osws'
task :run_all_osws do
  puts "Running all osws"
  run_osws(find_osws)
end

desc 'Run single osw measures ony'
task :run_osw_measures_only , [:workflow_name] do |task, args|
  args.with_defaults(workflow_name: 'bar_typical')
  workflow_name = args.workflow_name.inspect.delete('"')
  run_osw(workflow_name, true)
end

desc 'Setup and run single osw'
task :setup_run_osw , [:workflow_name] do |task, args|
  args.with_defaults(workflow_name: 'bar_typical')
  workflow_name = args.workflow_name.inspect.delete('"')
  setup_osw(workflow_name)
  run_osw(workflow_name)
end

desc 'Setup an analysis including zip file and OSA'
task :setup_osa , [:tbd_multiple_args] do |task, args|
  # todo update osw2osa to contain method that is called to make calling it with arguments cleaner
  # todo - update this to take multiple arguments
  # todo - consider running setup first or when know which OSW will be called
  system("ruby osw_2_osa.rb")
end

# todo - create task to meta CLI (there is more setup for computer for this to work provide good instructions)