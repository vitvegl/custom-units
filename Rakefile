module Systemd
  def daemon_reload
    system "systemctl daemon-reload"
  end

  def daemon_user_reload
    system "systemctl --user daemon-reload"
  end
end

module NetworkManager

  include Systemd

  UNIT='etc/systemd/system/NetworkManager.service'
  UNITDIR='/etc/systemd/system'
  UNITDST=File.join(UNITDIR, 'NetworkManager.service')

  def nm_unit_cp
    cp_r UNIT, UNITDST
  end

  def nm_enable_service
    sh "systemctl enable NetworkManager.service"
  end

  def nm_start_service
    sh "systemctl start NetworkManager.service"
  end

  def nm_restart_service
    sh "systemctl restart NetworkManager.service"
  end
end

module Iptables

  include Systemd

  RULES='etc/iptables/iptables.rules'
  RULESDIR='/etc/iptables'
  RULESDST='/etc/iptables/iptables.rules'
  UNIT='etc/systemd/system/iptables.service'
  UNITDST='/etc/systemd/system/iptables.service'

  def ipt_rules_cp
    mkdir RULESDIR unless Dir.exist? RULESDIR
    cp_r RULES, RULESDST
  end
  def ipt_unit_cp
    cp_r UNIT, UNITDST
  end

  def ipt_enable_service
    sh "systemctl enable iptables.service"
  end

  def ipt_start_service
    sh "systemctl start iptables.service"
  end

  def ipt_restart_service
    sh "systemctl restart iptables.service"
  end
end

module Ufw

  include Systemd

  def ufw_stop_mask
    ufw_flush_all
    system "for a in stop disable mask; do systemctl $a ufw.service; done"
  end

  private
  def ufw_flush_all
    system "for table in security raw mangle nat filter; do /sbin/iptables -t $table -F; done"
  end

  def ufw_pkg_remove
    ufw_stop_mask
    system "apt-get -y remove ufw"
  end
end

module Tor

  include Systemd

  USERUNITDIR = File.join(ENV['HOME'], '.config/systemd/user/')
  UNIT='home/user/.config/systemd/user/tor.service'
  UNITDST = File.join(USERUNITDIR, 'tor.service')

  def tor_pkg_install
    system "sudo apt-get update && sudo apt-get -y --no-install-recommends install tor"
  end

  def tor_unit_cp
    mkdir_p USERUNITDIR unless Dir.exist? USERUNITDIR
    cp_r UNIT, UNITDST
  end

  def tor_enable_service
    system "systemctl --user enable tor.service"
  end

  def tor_start_service
    system "systemctl --user restart tor.service"
  end

  def tor_system_stop_mask
    system "sudo systemctl disable tor.service"
    system "sudo systemctl stop tor.service"
    system "sudo systemctl mask tor.service"
  end
end

namespace :tor do
  include ::Tor

  desc 'Running Tor as service under non-privilegied user'
  task :service do
    if (Process.euid != 0 and Process.euid >= 1000)
      Rake::Task['tor:pkg_install'].invoke unless File.exist?('/usr/bin/tor')
      Rake::Task['tor:unit_cp'].invoke
      daemon_user_reload
      tor_enable_service unless File.symlink?(File.join(USERUNITDIR, 'network.target.wants/tor.service'))
      tor_start_service
    else
      puts "Please, run this task as non-root user"
    end
  end

  task :pkg_install do
    tor_pkg_install
    tor_system_stop_mask
  end

  task :unit_cp do
    tor_unit_cp
  end
end

namespace :iptables do
  include ::Iptables

  task :rules do
    ipt_rules_cp
  end
  task :unit do
    ipt_unit_cp
  end

  desc 'copying iptables rules and starting service'
  task :service => ['ufw:stop', 'rules', 'unit'] do
    Rake::Task['ufw:removepkg'].invoke
    ipt_enable_service
    ipt_start_service
  end
end

namespace :ufw do

  include ::Ufw
  desc 'stopping and mask ufw as service'
  task :stop do
    ufw_stop_mask
  end

  task :removepkg do
    ufw_pkg_remove
  end
end

namespace :nm do
  include ::NetworkManager
  namespace :xenial do
    desc 'NetworkManager service for Ubuntu 16.04 LTS (apparmor fix)'
    task :service do
      nm_unit_cp unless File.exist? UNITDST
      nm_enable_service unless File.symlink?(File.join(UNITDIR, 'multi-user.target.wants/NetworkManager.service'))
      daemon_reload
      nm_restart_service
    end
  end
end
