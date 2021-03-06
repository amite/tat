require 'yaml'
require 'sshkit'
require 'sshkit/dsl'

config = YAML.load_file('_config.yml')

deploy_server = config['deploy_server']
deploy_path = config['deploy_path']
deploy_user = config['deploy_user']


SSHKit::Backend::Netssh.configure do |ssh|
  ssh.ssh_options = {
    user: deploy_user,
    auth_methods: ['publickey']
  }
end

class Site
  def initialize(host, path, owner)
    @host = host
    @path = path
    @owner = owner
  end

  def deploy!
    compile
    upload
  end

  private

  def upload
    with_deploy_location_ready do
      @host.upload! "_site/", @path, recursive: true
    end
  end

  def with_deploy_location_ready
    create_deploy_path unless deploy_path_exists?
    chown_to_owner unless deploy_path_owned?
    yield
  end

  def deploy_path_exists?
    @host.test "[ -d #{@path} ]"
  end

  def create_deploy_path
    @host.as @owner do
      @host.execute 'mkdir', '-p', @path
    end

  end

  def deploy_path_owned?
    @host.test "[ `stat -c %U #{@path}` = '#{@owner}']"
  end

  def chown_to_owner
    @host.as @owner do
      @host.execute 'chown', '-R', @owner, @path
    end
  end

  def compile
    # @host.run_locally { execute 'jekyll', 'build' }
  end

end

desc 'Deploy the site to production'
task :deploy do
  on deploy_server do
    Site.new(self, deploy_path, deploy_user).deploy!
  end
end




# SSHKit.config.output_verbosity = :debug

# host = config['host']
# site_dir = config['sitedir']

# task :echo  do
#   run_locally do
#     execute "echo 'hi'"
#   end
# end



# desc 'Who am I on the server'
# task :who do
#   on host do
#     as 'www-data' do
#       puts capture(:whoami)
#     end
#   end
# end

# desc  'upload a file to the server'
# task :upload_file do
#   on host do
#     file = File.open('sample.txt')
#     upload! file, "#{site_dir}/sample.txt"
#   end
# end