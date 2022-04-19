
# Ansible Role: Openstreetmap

Installs and configure a fully working OpenStreetMap mirror based on PG 14, PostGIS 3.2 and renderd

## Requirements
### Ansible modules
This role require **posix** ansible module, so you may have to launch : `ansible-galaxy collection install ansible.posix`

### Privileged access
This role requires root access, so either run it in a playbook with a global `become: yes`, or invoke the role in your playbook like:

    - hosts: osm
      roles:
        - role: openstreetmap    
          become: yes

### Hardware requirements
The whole planet extract take around **1.8To** while imported into database. For this reason, please make sure you have enough space into **/var/lib/postgresql** (database data directory) and **/home/osm** (main directory) 

In order to work great, you will need really fast disks (**SSD**) and as many ram as possible (at least **128Go RAM** for a planet mirror). RAM is really important for this software, as much you can have, the better your OSM mirror will work

There is not that much informations on the web, but for further informations about hardware requirements, you can check https://hardware.openstreetmap.org/. It is a list of nodes hosted for openstreetmap.org organization

## Role Variables
The following variables are always required:

	tileserver_main_domain: osm.imajing.fr
	tileserver_subdomains:
	  - a.osm.imajing.fr
	  - b.osm.imajing.fr
	  - c.osm.imajing.fr
	  - d.osm.imajing.fr
	leaflet_address: "osm.imajing.fr"

  Note than you can get a **full list of available variables** by checking **`example/vars.yml`** file  

## Example Playbook
    - hosts: servers
      roles:
        - role: mpaletou.openstreetmap 
          become: true
      vars:
        tileserver_main_domain: osm.imajing.fr
        tileserver_subdomains:
          - a.osm.imajing.fr
          - b.osm.imajing.fr
          - c.osm.imajing.fr
          - d.osm.imajing.fr
        leaflet_address: "osm.imajing.fr"
        # By default, the whole planet extract is imported which is really big.
        # You can get zone extracts address here: https://download.openstreetmap.fr/extracts/
        osm_map_pbf_url: https://download.openstreetmap.fr/extracts/oceania-latest.osm.pbf

### Default versions
- osm2pgsql 1.4.1 (1.6.0 available is latest repo). You can get further informations about available version from https://github.com/openstreetmap/osm2pgsql
- mod_tile/renderd (0.6.1) You can get further informations about available version from https://github.com/openstreetmap/mod_tile

## Specific informations

### Mod_tile (renderd) configuration

You can personalize *mod_tile* configuration if you want to customize:
- tile cache limits
- renderd request limits
- define timeouts (set by default to 30s)

In order to do that, you only have to edit `templates/apache2/tileserver.vhost.j2`

By default, most of those parameters are disabled or set to huge value (no limit) for a better operation. However, defining some limitations can be useful if your server is not very powerful

### Mirror updates
There is two ways to update your mirror:
- **automatic way**: define `osm_replication: true` to configure osm2pgsql-replication. This service will auto-synchronize your mirror with `osm_replication_server`
By default, it uses https://planet.openstreetmap.org/replication/minute so you will get minutly updates. You need to use **osm2pgsql 1.6.0+** to make this feature work
- **manual way**: use `runuser -u osm -- osm2pgsql --append` command like described [here](https://osm2pgsql.org/doc/manual.html#command-line-options)

### Leaflet
By default, this playbook installs Leaftlet which is a web utility able to browse tiles rendered by your server. Without specific configuration, it is made available from: *http://<your_leaflet_address>:8080*

### Tiles prerendering:

If you have [a lot a space on your server](https://wiki.openstreetmap.org/wiki/Tile_disk_usage) and you would like to prerender tiles, you can launch the following commands:

##### Launch import process in the foreground

    render_list -m default -a -z 0 -Z 19 --num-threads=10 --force

##### Launch import process in the background

    render_list -m default -a -z 0 -Z 19 --num-threads=10 &

For further informations, you can refer to: https://wiki.openstreetmap.org/wiki/Tile_disk_usage

## License

MIT / BSD

## Author Information

This role was created in 2022 by [Malo Paletou](https://github.com/mpaletou), for [Imajing SAS purpose](https://imajing.eu/).
