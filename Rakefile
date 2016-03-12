module Iptables
  RULES='etc/iptables/iptables.rules'
  RULESDIR='/etc/iptables'
  RULESDST='/etc/iptables/iptables.rules'
  UNIT='etc/systemd/system/iptables.service'
  UNITDST='/etc/systemd/system/iptables.service'

  def ipt_rules_cp
    Dir.mkdir RULESDIR unless Dir.exist? RULESDIR
    FileUtils.cp_r RULES, RULESDST
  end
  def ipt_unit_cp
    FileUtils.cp_r UNIT, UNITDST
  end
end

module Ufw
  def ufw_stop_mask
    ufw_flush_all
    system "for a in stop disable mask; do systemctl $a ufw.service; done"
  end

  private
  def ufw_flush_all
    system "for table in security raw mangle nat filter; do /sbin/iptables -t $table -F; done"
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
    sh "systemctl enable iptables && systemctl restart iptables"
  end
end

namespace :ufw do

  include ::Ufw
  desc 'stopping and mask ufw as service'
  task :stop do
    ufw_stop_mask
  end
end
