# Đổi tên Region, Availability Zone trong openstack

1. ### Đổi tên Region

   - **Region**: Là một thực thể API v3 dịch vụ nhận dạng, đại diện cho một bộ phận chung trong triển khai Openstack. Có thể tạo và liên kết chúng với các region phụ để tạo cấu trúc dạng cây. Region không có ý nghĩa về mặt địa lý nhưng tên của region thường được đặt theo tên khu vực địa lý(vd: HaNoi, SaiGon)

   - **Service**: Một dịch vụ Openstack, như Compute (nova), Network(neutron), Block storage(cinder), hay Image service (glance), là một hay nhiều endpoint mà qua đó người dùng có thể truy cập tài nguyên và thực hiện các hành động.

   - **Endpoint**: Là một địa chỉ truy cập mạng, thường là địa chỉ URL của một dịch vụ để từ đó truy cập vào dịch vụ.

     1. Đổi tên Region Keystone

        - Lấy id_endpoint từ danh sách endpoint:

          ```
          openstack endpoint list
     
        - Set new region cho endpoint ( cả 3 interface Internal,admin,public):

          ```
          openstack endpoint set --region New_name_Region ID_endpoint
     
        - Kiểm tra lại bằng cách xem có lấy được token hay không: 

          ```
          openstack token issue
     
     2. Đổi tên Region Glance

        - Lấy id_endpoint từ danh sách endpoint: 

          ```
          openstack endpoint list
     
        - Set new region cho endpoint ( cả 3 interface Internal,admin,public của service **Keystone**): 

          ```
          openstack endpoint set --region New_Name_Region ID_endpoint
     
        - Thêm trong file config glance-api.conf: 

          ```
          [keystone_authtoken]
          region_name = New_name_region
          [oslo_limit]
          region_name = New_name_region
     
        - Populate the Image service database:

          ```
          su -s /bin/sh -c "glance-manage db_sync" glance
     
        - Khởi động lại dịch vụ glance: 

          ```
          service glance-api restart
     
     3. Đổi tên Region Nova

        - Lấy id_endpoint từ danh sách endpoint: 

          ```
          openstack endpoint list
     
        - Set new region cho endpoint ( cả 3 interface Internal,admin,public của cả service **Placement và Nova** ): 

          ```
          openstack endpoint set --region New_Name_Region ID_endpoint
     
        - Thêm trong file config placement.conf (cả các node controller và compute):

          ```
          [keystone_authtoken]
          region_name = New_name_region
     
        - Thêm trong file config nova.conf (cả các node controller và compute): 

          ```
          [keystone_authtoken]
          region_name = New_name_region
          [placement]
          region_name = New_name_region
          ```
     
        - Populate the `placement` `nova` database: 

          ```
          su -s /bin/sh -c "nova-manage db sync" nova
          ```
     
          ```
          su -s /bin/sh -c "placement-manage db sync" placement
          ```
     
        - Khởi động lại các dịch vụ Placement và Nova: 

          ```
          service nova-api restart
          ```
     
          ```
          service apache2 restart
          ```
     
          ```
          service nova-compute restart
     
     4. Đổi tên Region Neutron

        - Lấy id_endpoint từ danh sách endpoint: 

          ```
          openstack endpoint list
          ```
     
        - Set new region cho endpoint ( cả 3 interface Internal,admin,public của service **Neutron** ): 

           ```
           openstack endpoint set --region New_Name_Region ID_endpoint
           ```
     
        - Thêm trong file config neutron.conf (cả các node controller và network):

          ```
          [keystone_authtoken]
          region_name = New_name_region
     
        - Sửa region_name trong file nova.conf (cả các node controller và compute):

          ```
          [neutron]
          region_name = HaNoi
     
        - Populate the database:

          ```
          su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
            --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
     
        - Khởi động lại dịch vụ Neutron và Nova: 

          ```
          service nova-api restart
          ```
     
          ```
          service neutron-server restart
     
     5. Đổi tên Region Cinder

        - Lấy id_endpoint từ danh sách endpoint: 

          ```
          openstack endpoint list
     
        - Set new region cho endpoint ( cả 3 interface Internal,admin,public của service **cinder** ): 

          ```
          openstack endpoint set --region New_Name_Region ID_endpoint
          ```
     
        - Thêm trong file config cinder.conf (cả các node controller và storage):

          ```
          [keystone_authtoken]
          region_name = New_name_region
          ```
     
        - Thêm trong file config nova.conf (cả các node controller và compute): 

          ```
          [cinder]
          os_region_name = New_name_region
          ```
     
        - Populate the `cinder` database: 

          ```
          su -s /bin/sh -c "cinder-manage db sync" cinder
          ```
     
        - Khởi động lại các dịch vụ `cinder` và `Nova`: 

          ```
          service nova-api restart
          ```
     
          ```
          service cinder-scheduler restart
          ```
     
          ```
          service apache2 restart
          ```
     
     6. Verify 

        - Kiểm tra xem danh sách region: 

          ```
          root@controller-node1:~# openstack region list
          /usr/local/lib/python3.8/dist-packages/pkg_resources/__init__.py:116: PkgResourcesDeprecationWarning: 0.23ubuntu1 is an invalid version and will not be supported in a future release
            warnings.warn(
          +--------+---------------+-------------+
          | Region | Parent Region | Description |
          +--------+---------------+-------------+
          | HaNoi  | None          |             |
          +--------+---------------+-------------+
     
        - Kiểm tra danh sách endpoint:

          ```
          root@controller-node1:~# openstack endpoint list
          /usr/local/lib/python3.8/dist-packages/pkg_resources/__init__.py:116: PkgResourcesDeprecationWarning: 0.23ubuntu1 is an invalid version and will not be supported in a future release
            warnings.warn(
          +----------------------------------+--------+--------------+--------------+---------+-----------+------------------------------------------+
          | ID                               | Region | Service Name | Service Type | Enabled | Interface | URL                                      |
          +----------------------------------+--------+--------------+--------------+---------+-----------+------------------------------------------+
          | 069a64602a084dcf85cb0d033fe7c0da | HaNoi  |              | image        | True    | internal  | http://controller:9292                   |
          | 0ed44072f02b4cefa5dc68c9281b722f | HaNoi  | neutron      | network      | True    | public    | http://controller:9696                   |
          | 1affc0c7f82f416eb90be4d7ef4f9083 | HaNoi  | nova         | compute      | True    | internal  | http://controller:8774/v2.1              |
          | 391a39af0e0042de959fd2ddb3330d3e | HaNoi  | nova         | compute      | True    | public    | http://controller:8774/v2.1              |
          | 3e15cdfedb4a47f6bdcd15f6e2d6f1e1 | HaNoi  |              | image        | True    | public    | http://controller:9292                   |
          | 4919213bf4db459c86401e2f501a3add | HaNoi  |              | image        | True    | admin     | http://controller:9292                   |
          | 4936edf3449c4769975ca534572deb8f | HaNoi  | cinderv2     | volumev2     | True    | internal  | http://controller:8776/v2/%(project_id)s |
          | 5563aa0d62024be2a6e6199ba8f9a3f3 | HaNoi  | keystone     | identity     | True    | public    | http://controller:5000/v3/               |
          | 6450b09f3c204930990d65bc3b9ec3de | HaNoi  | cinderv3     | volumev3     | True    | internal  | http://controller:8776/v3/%(project_id)s |
          | 6cefc1f2fe6c4a92bd7785d8c1a32f0c | HaNoi  | neutron      | network      | True    | admin     | http://controller:9696                   |
          | 77b81843cbba490a9174ea37b9bac477 | HaNoi  | keystone     | identity     | True    | public    | http://controller:5000/v3/               |
          | 7d2623fd753e4d6b927269f8713484c4 | HaNoi  | cinderv3     | volumev3     | True    | public    | http://controller:8776/v3/%(project_id)s |
          | 9f818e4eedc04a72b9462803a8b31dda | HaNoi  | placement    | placement    | True    | internal  | http://controller:8778                   |
          | a42d283fa6e44ee09e9678a3580224bb | HaNoi  | cinderv2     | volumev2     | True    | public    | http://controller:8776/v2/%(project_id)s |
          | a539cf6fbc324341b0afb28702efb8da | HaNoi  | nova         | compute      | True    | admin     | http://controller:8774/v2.1              |
          | a940b851b2134433aec1259a788c9e9b | HaNoi  | neutron      | network      | True    | internal  | http://controller:9696                   |
          | c6ece1d66a3b423abe8d7b1d62bd45ab | HaNoi  | keystone     | identity     | True    | admin     | http://controller:5000/v3/               |
          | e5d04f7260a548af821c783ae2dc03a5 | HaNoi  | placement    | placement    | True    | admin     | http://controller:8778                   |
          | ed817344977c47bdac2f956a974868e3 | HaNoi  | cinderv3     | volumev3     | True    | admin     | http://controller:8776/v3/%(project_id)s |
          | f2ae784d77ad4ccb87016bb21fa9213b | HaNoi  | cinderv2     | volumev2     | True    | admin     | http://controller:8776/v2/%(project_id)s |
          | f5c2e83cbddb490e880ee043b5dcb10c | HaNoi  | keystone     | identity     | True    | internal  | http://controller:5000/v3/               |
          | fe01e71506774218afdbfad960e6905b | HaNoi  | placement    | placement    | True    | public    | http://controller:8778                   |
          +----------------------------------+--------+--------------+--------------+---------+-----------+------------------------------------------+
   
2. ### Đổi tên Availability Zone

   Trong một region cần phân biệt các Availability Zone để phân biệt và quản lí dễ dàng. 
   
   ![Openstack-Zoning](https://kimizhang.files.wordpress.com/2013/08/openstack-zoning.jpg?w=590)
   
   1. Đổi tên AZ của Compute
   
      ![host aggregates](https://sp-ao.shortpixel.ai/client/q_glossy,ret_img,w_194,h_189/https://www.mirantis.com/wp-content/uploads/2017/01/hostaggregates.png)
   
      - Tạo 1 Host Aggregate (nên đặt tên để phân loại mục đích) 
   
        ```
        openstack aggregate create name_aggregate
   
      - Tạo Availability Zone và liên kết nó với Host Aggregate vừa tạo
   
        ```
        openstack aggregate set –zone <az_name>  <host_aggregate_name>
   
      - Thêm compute host vào nhóm Host Aggregate
   
        ```
        openstack aggregate add host <host_aggregate_name>  <compute_host>
   
      - Cấu hình AZ default nếu không chỉ định AZ trong lệnh gọi API (config trong nova.conf ở controller node)
   
        ```
        [DEFAULT]
        default_schedule_zone=AZ_name
   
      - Populate the `nova` database: 
   
        ```
        su -s /bin/sh -c "nova-manage db sync" nova
        ```
   
      - Khởi động lại dịch vụ nova-api
   
        ```
        service nova-api restart
   
      - Verify 
   
        ```
        root@controller-node1:~# openstack availability zone list --compute
        /usr/local/lib/python3.8/dist-packages/pkg_resources/__init__.py:116: PkgResourcesDeprecationWarning: 0.23ubuntu1 is an invalid version and will not be supported in a future release
          warnings.warn(
        +-----------+-------------+
        | Zone Name | Zone Status |
        +-----------+-------------+
        | HaNoi-1   | available   |
        | internal  | available   |
        +-----------+-------------+
   
   2. Đổi tên AZ của Volume
   
      - Cấu hình vào cinder.conf trên các node cinder-volume:
   
        ```
        [DEFAULT]
        storage_availability_zone=AZ_name
   
      - Cấu hình vào cinder.conf trên các node cinder-api:
   
        ```
        [DEFAULT]
        default_availability_zone=AZ_name
        allow_available_zone_fallback = True //
   
      - Để cấu hình instance nhận volume từ AZ cùng tên cần cấu hình nova.conf ở các nút compute:
   
        ```
        [cinder]
        cross_az_attach = False
   
      - Tuy nhiên các volume cũ vẫn đang nhận AZ là nova, cần thay đổi trong database:
   
        ```
        use cinder;
        update set availability_zone='AZ_name' where availability_zone='nova';
   
      - Populate the `cinder` database: 
   
        ```
        su -s /bin/sh -c "cinder-manage db sync" cinder
        ```
   
      - Khởi động lại dịch vụ:
   
        ```
        service cinder-volume restart
        service cinder-scheduler restart
        ```
   
      - verify
   
        ```
        root@controller-node1:~# openstack availability zone list --volume
        /usr/local/lib/python3.8/dist-packages/pkg_resources/__init__.py:116: PkgResourcesDeprecationWarning: 0.23ubuntu1 is an invalid version and will not be supported in a future release
          warnings.warn(
        +-----------+-------------+
        | Zone Name | Zone Status |
        +-----------+-------------+
        | HaNoi-1   | available   |
        +-----------+-------------+
   
   3. Đổi tên AZ của Network
   
      - Cấu hình dhcp_agent.ini và l3_agent.ini
   
        ```
        [AGENT]
        Availability_zone = AZ_name
   
      - Cấu hình neutron.conf trên node neutron-server:
   
        ```
        [DEFAULT]
        default_availability_zones=AZ_name
   
      - Cấu hình trong nova.conf(tất cả các node compute) để sửa lỗi mạng VIF khi tạo máy ảo
   
        ```
        vif_plugging_is_fatal=false
        vif_plugging_timeout=0
   
      - Populate the database:
   
        ```
        su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
          --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
        ```
   
      - Khởi động lại dịch vụ:
   
        ```
        service neutron-server restart
        service nova-compute restart
   
      - verify:
   
        ```
        root@controller-node1:~# openstack availability zone list --network
        /usr/local/lib/python3.8/dist-packages/pkg_resources/__init__.py:116: PkgResourcesDeprecationWarning: 0.23ubuntu1 is an invalid version and will not be supported in a future release
          warnings.warn(
        +-----------+-------------+
        | Zone Name | Zone Status |
        +-----------+-------------+
        | HaNoi-1   | available   |
        | HaNoi-1   | available   |
        +-----------+-------------+

##### Sau khi hoàn thành việc đổi Region và AZ mọi tính năng hoạt động bình thường, các dữ liệu cũ không bị ảnh hưởng
