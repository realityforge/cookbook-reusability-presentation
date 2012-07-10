!SLIDE
# Reusable Chef Cookbooks
## The Evolution

!SLIDE

# Phase 1
## Cookbook as a big bash script

!SLIDE smaller

# Example #

	@@@ ruby
    bash "install mypackage" do
      cwd "#{Chef::Config[:file_cache_path]}"
      code <<-EOH
    wget http://example.com/mypackage-1.0.tar.gz
    tar xzf mypackage-1.0.tar.gz
    cd mypackage-1.0
    ./configure && make && make install
      EOH
      not_if { File.exists?("/usr/bin/mypackage") }
    end

!SLIDE

# Pros

* Fast and Easy

!SLIDE

# Cons

* Everything else

!SLIDE

# Phase 2
## Customize using attributes

!SLIDE smaller smaller

# Example #

	@@@ ruby
	filename =
	  "mypackage-#{node[:mypackage][:version]}.tar.gz"
    bash "install mypackage" do
      cwd "#{Chef::Config[:file_cache_path]}"
      code <<-EOH
    wget http://example.com/#{filename}
    tar xzf #{filename}
    cd mypackage-#{node[:mypackage][:version]}
    ./configure && make && make install
      EOH
      not_if { File.exists?("/usr/bin/mypackage") }
    end

    template "/etc/mypackage.conf" do
      source "mypackage.conf.erb"
      mode "0644"
      variables(
          :database => node[:mypackage][:database],
          :user => node[:mypackage][:user],
          :password => node[:mypackage][:password]
        )

!SLIDE

# Pros

* Easy to customize environment specific settings

!SLIDE

# Cons

* Does not allow customization of components installed

!SLIDE

# Phase 3
## Partition recipes into units of reuse

!SLIDE smaller smaller

# Example #

	@@@ ruby
	include_recipe "mypackage::default"
	# Include different recipes based on
	# nodes characteristics
	if node['datacenter'] != 'BWD'
	  include_recipe "mypackage::db_auth"
	else
	  include_recipe "mypackage::ad_auth"
	end

!SLIDE

# Pros

* Easy to customize the install for different nodes

!SLIDE

# Cons

* Can not loop over recipes for repetition

!SLIDE

# Phase 4
## Abstract repeated actions using resources

!SLIDE smaller smaller

# Example #

	@@@ ruby
    glassfish_mq_destination "WildfireStatus queue" do
      queue "Fireweb.WildfireStatus"
      config {'validateXMLSchemaEnabled' => true,
              'XMLSchemaURIList' => 'http://...'}
      host 'localhost'
      port 7676
    end

!SLIDE smaller smaller

# Composable Example #

	@@@ ruby
    glassfish_mq "MessageBroker Instance" do
      instance "MessageBroker"
      users {...}
      access_control_rules {...}
      config {...}
      queues {
        "Fireweb.WildfireStatus" =>
                {'validateXMLSchemaEnabled' => true,
                 'XMLSchemaURIList' => 'http://...'}
      }
      port 7676
      admin_port 7677
      jms_port 7678
      jmx_port 8087
      stomp_port 8088
    end

!SLIDE

# Pros

* Reduce repetition of similar code
* Composable abstractions
* Simplify the cognitive load

!SLIDE

# Cons

* Still repetitive if you need to define 30 queues
* Multiple products require updates to a single cookbook

!SLIDE

# Phase 5
## Data driven reuse

!SLIDE

* synthesize using search
* load from data bags or external data store
* synthesize using rule layer

!SLIDE smaller smaller smaller

# Search Example #

	@@@ ruby
	queues = []
    search(:node, 'omq_dests_queues:*' + node.name) do |n|
      queues.merge( n['omq']['dests']['queues'].to_hash )
    end
    queues.merge( node['omq']['dests']['queues'].to_hash )

    queues.each_pair do |key, value|
      glassfish_mq_destination key do
        queue key
        config value
        host 'localhost'
        port 7676
      end
    end

!SLIDE

# Pros

* Reduce typing to minimal set

!SLIDE

# Cons

* Mixes policy into cookbook

!SLIDE

# Phase 6
## Separate policy into paired cookbook

!SLIDE

* Derive data in cookbook A
* Set attributes on the node
* include recipe in cookbook B

!SLIDE smaller smaller smaller

# Example #

	@@@ ruby
    search(:node, 'omq_dests_queues:*' + node.name) do |n|
      n.to_hash.each_pair do |key, value|
        node.override['omq']['dests']['queues'][key] = value
      end
    end
    include_recipe "glassfish::attribute_driven_mq"

!SLIDE

# Phase 7
## Standardize policy, Customize using attributes

!SLIDE   smaller smaller

# Example #

	@@@ ruby
    node.override['graphite']...['schema_search']['filter'] =
      "monitor_environment:#{node.chef_environment}"
    include_recipe "graphite::search_based"


!SLIDE
# Peter Donald
## peter at realityforge.org,
## <span class="callout">@rahvintaka</span>

