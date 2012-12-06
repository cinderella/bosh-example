# BOSH Notes

In an effort to save time with downloading and uploading artifacts, the following steps were all done on an EC2 instance.

Because BOSH documentation is sparse and out of date, I've compiled these notes to help with future work.

## Assumptions
- ubuntu with ruby installed
- install keypair for use with github

## Procedure

0. forked `cloudfoundry/bosh` to `cinderella/bosh` in order to make changes necessary for Cinderella.
1. make changes and commit to `cinderella/bosh`
2. cd ..
3. Check out bosh-release: `git clone https://github.com/cloudfoundry/bosh-release.git`
4. cd bosh-release
5. `git submodule update --init`
6. `bosh create release --with-tarball` (will create manifest and tarball in bosh-release/dev_releases needed in step 10)
7. cd ../bosh/agent
8. bundle install
9. extra packages needed: `sudo apt-get install debootstrap kpartx`
10. create stemcell from manifest and tarball created in step 6: `rake stemcell2:micro[aws,/root/projects/bosh-release/dev_releases/cinders-10.1-dev.yml,/root/projects/bosh-release/dev_releases/cinders-10.1-dev.tgz]`
  
	Generated stemcell: /var/tmp/bosh/agent-0.6.7-18060/work/work/micro-bosh-stemcell-aws-0.7.0.tgz

11. create inception vm: https://github.com/drnic/bosh-getting-started/blob/master/create-a-bosh/aws/create-an-aws-inception-vm.md
12. create micro bosh from new stemcell: https://github.com/drnic/bosh-getting-started/blob/master/create-a-bosh/creating-a-micro-bosh-from-stemcell.md

---

## References

- Prepare inception script: https://raw.github.com/drnic/bosh-getting-started/master/scripts/prepare_inception.sh
- Hints on how to create a cpi: https://groups.google.com/a/cloudfoundry.org/forum/#!msg/bosh-dev/vqu_uqdb8Wo/021IPrRtizUJ

---


## Using Fog to Create an Inception VM

```
connection = Fog::Compute.new({ :provider => 'AWS', :region => 'us-east-1' })

Fog.credentials = Fog.credentials.merge({ :private_key_path => "/Users/shane/.ssh/fog_rsa", :public_key_path => "/Users/shane/.ssh/fog_rsa.pub" })
connection.import_key_pair('shane-fog', IO.read('/Users/shane/.ssh/fog_rsa.pub')) if connection.key_pairs.get('shane-fog').nil?

server = connection.servers.bootstrap({:key_name => 'shane-fog', :flavor_id => 'm1.small', :bits => 64, :username => 'ubuntu'})

address = connection.addresses.create
address.server = server
server.reload
server.dns_name

volume = connection.volumes.create(:size => 16, :device => "/dev/sdi", :availability_zone => server.availability_zone)
volume.server = server

server.ssh(['sudo mkfs.ext4 /dev/sdi -F'])
server.ssh(['sudo mkdir -p /var/vcap/store'])
server.ssh(['sudo mount /dev/sdi /var/vcap/store'])
puts server.ssh(['df']).first.stdout

ssh -i /Users/shane/.ssh/fog_rsa ubuntu@ec2-174-129-251-149.compute-1.amazonaws.com
sudo su -

export ORIGUSER=ubuntu
curl -s https://raw.github.com/drnic/bosh-getting-started/master/scripts/prepare_inception.sh | bash
source /etc/profile


create security group for micro bosh

connection = Fog::Compute.new({ :provider => 'AWS', :region => 'us-east-1' })
connection.security_groups.create name: "microbosh", description: "microboshes"
group = connection.security_groups.get("microbosh")
group.authorize_port_range(25555..25555) # BOSH Director API
group.authorize_port_range(6868..6868)   # Message Bus
group.authorize_port_range(25888..25888) # AWS Registry API
group.authorize_port_range(22..22)       # SSH access
```