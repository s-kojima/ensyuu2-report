#�ץ���������
�ơ��֥빽���ϰʲ��˼����ơ��֥빽���Ȥ�����![�ơ��֥빽��](https://github.com/s-kojima/ensyuu2-report/blob/master/table.png)
�������٤�ư���ͤ��뤿�ᡤhost1����host2��ping���������ͤ��롥
host1����host2��ping������Ȥޤ�host1����롼���˥ѥ��åȤ����롥
���ΤȤ���host1�ϥ롼���˥ѥ��åȤ�����٤ˤޤ�ARP�ꥯ�����Ȥ��������롥
³���ƥ롼���Ϥ�����Ф���������Ԥ���host1�Ϥ��ξ���򸵤�
�ѥ��åȤ��������롥�롼����host1���餭���ѥ��åȤ�host2���Ϥ���Ȥ�
host2�ΰ���MAC���ɥ쥹��ʬ����ʤ��Τǡ�ARP̤���ʥѥ��åȤȤ���
����ȥ���ˤ���Ƥ��������θ�ARP�ꥯ�����Ȥ�host2���Ф�����������
ARP��ץ饤���֤äƤ���ARP����褵�줿�塤����ȥ���ˤ���Ƥ�����
�ѥ��åȤ���������Ф褤���ʾ��դޤ��ơ��ޤ��Ϥ�����ɲä��Ƥ����٤�
�ե�����ȥ��Ҥ٤롥

Protocol Classifier�ơ��֥뤬�ѥ��åȤμ���(ARP��IPv4��)�ˤ�ä�
ARP RESPONDER��Routing Table�Τɤ���Υơ��֥�˰�ư���뤫��
Ƚ�̤��롥
���Τ��ᡤClassifier�ơ��֥�˼��Τ褦�ˤ��ơ��ե�����ȥ��Ϥ�����ɲä��롥
```
    send_flow_mod_add(
      dpid,
      table_id: CLASSIFIER_TABLE_ID,
      idle_timeout: 0,
      priority: 0,
      match: Match.new(
	ether_type: ETHER_TYPE_ARP
      ),
      instructions: GotoTable.new(ARP_RESPONDER_TABLE_ID)
    )

    send_flow_mod_add(
      dpid,
      table_id: CLASSIFIER_TABLE_ID,
      idle_timeout: 0,
      priority: 0,
      match: Match.new(ether_type: ETHER_TYPE_IPv4),
      instructions: GotoTable.new(ROUTING_TABLE_ID)
    )
```
ARP RESPONDER�ơ��֥�Ǥ�ARP�ѥ��åȤ˴ؤ��������Ԥ���
ARP�ꥯ�����Ȥ�ȯ��������硤OpenFlow1.0�ǥ롼���Ǥ�packet in��ȯ������
������ԤäƤ��������롼���ϼ��ȤΥݡ��ȤȤ�����б�����MAC���ɥ쥹���ΤäƤ��뤿��packet in�򵯤����ʤ��Ƥ⡤arp request���֤����Ȥ��Ǥ��롥
�롼�����Ȥξ���ϼ��Τ褦�ˤ�����������Ǥ��롥
```
    interface_hash = Configuration::INTERFACES
```
Configuration::INTERFACES�ϥϥå��幽¤�ˤʤäƤ��ꡤ�롼�����
�ݡ��ȤȤ�����б�����MAC���ɥ쥹�ʤɤ����줾�����äƤ��롥
ARP�ꥯ�����Ȥ��褿�Ȥ������Υꥯ�����Ȥ�ARP��ץ饤���Ѥ���
ARP�ꥯ�����Ȥ�������MAC���ɥ쥹�ϼ���������MAC���ɥ쥹�ˤʤ롥
�ޤ���ARP��ץ饤��������MAC���ɥ쥹(�롼���Υݡ��Ȥ��б�����MAC���ɥ쥹)��
IP���ɥ쥹���ʤɤ�Ʊ�������ꤹ��ɬ�פ����롥������Υݡ����ֹ��
Ʊ���ݡ��Ȥ˽Ф��Τǡ�reg1�����򤵤��Ƥ�����
�����Υ��������ȡ��ѥ��åȤ�������Ԥ�EGRESS�ơ��֥�ؤΰ�ư��
���Τ褦�˥��åȤ��롥
```
    ip_address = IPv4Address.new(each.fetch(:ip_address))
    actions = [
           NiciraRegMove.new(from: :source_mac_address,
                             to: :destination_mac_address),
	   SetSourceMacAddress.new(each.fetch(:mac_address)),
	   SetArpOperation.new(Arp::Reply::OPERATION),
	   NiciraRegMove.new(from: :arp_sender_hardware_address,to: :arp_target_hardware_address),
          NiciraRegMove.new(from: :arp_sender_protocol_address,to: :arp_target_protocol_address),
           SetArpSenderHardwareAddress.new(each.fetch(:mac_address)),
	   SetArpSenderProtocolAddress.new(ip_address),
           NiciraRegLoad.new(value, :in_port),
           NiciraRegLoad.new(each.fetch(:port), :reg1)
	]

    send_flow_mod_add(
       dpid,
       table_id: ARP_RESPONDER_TABLE_ID,
       idle_timeout: 0,
       priority: 0,
       match: Match.new(
              ether_type: ETHER_TYPE_ARP,
              arp_operation: Arp::Request::OPERATION,
              arp_target_protocol_address: ip_address,
              dst_protocol_address:ip_address,
              in_port: each.fetch(:port)),
       instructions: [Apply.new(actions),GotoTable.new(EGRESS_TABLE_ID)])
```

�롼�������ä�ARP�ꥯ�����Ȥ��Ф���ARP��ץ饤���褿�Ȥ���
����ȥ�����䤤��碌�뤿�ἡ�Υե�����ȥ���ɲä��Ƥ�����
```
    send_flow_mod_add(
       dpid,
       table_id: ARP_RESPONDER_TABLE_ID,
       idle_timeout: 0,
       priority: 0,
       match: Match.new(
              ether_type: ETHER_TYPE_ARP,
              arp_operation: Arp::Reply::OPERATION,
              arp_target_protocol_address: ip_address,
              dst_protocol_address:ip_address,
              in_port: each.fetch(:port)),
       instructions: Apply.new(SendOutPort.new(:controller))
     )
```
Routing�ơ��֥�Ǥ�ipv4�ѥ��åȤ������襢�ɥ쥹��
�ݻ����뤿�ᡤreg0�������襢�ɥ쥹�����򤵤��롥
```
    actions = [NiciraRegMove.new(from: :ipv4_destination_address,to: :reg0)]
    send_flow_mod_add(
       dpid,
       table_id: ROUTING_TABLE_ID,
       idle_timeout: 0,
       priority: 10,
       match: Match.new(
              ether_type: ETHER_TYPE_IPv4,
              ipv4_destination_address: ip_address_mask,
              dst_protocol_address:ip_address,
              ipv4_destination_address_mask: default_mask2),
       instructions: [Apply.new(actions),GotoTable.new(INTERFACE_LOOKUP_TABLE_ID)]
     )   
```

�ޤ�������INTERFACE LOOKUP�ơ��֥�Ǥ�
�������뤿��Υݡ����ֹ��reg1�����򤷤Ƥ�������������MAC���ɥ쥹��
�롼���Τ�Τ˽񤭴����롥�����Ƽ���ARP LOOKUP�ơ��֥�˰�ư���뤿�ᡥ
���Τ褦�˥ե�����ȥ���ɲä��롥
```
    actions = [NiciraRegLoad.new(each.fetch(:port),:reg1),
               SetSourceMacAddress.new(each.fetch(:mac_address))]
    send_flow_mod_add(
       dpid,
       table_id: INTERFACE_LOOKUP_TABLE_ID,
       idle_timeout: 0,
       priority: 0,
       match: Match.new(
              reg0: ip_address_mask.to_i,
              reg0_mask: default_mask2.to_i),
       instructions: [Apply.new(actions),GotoTable.new(ARP_LOOKUP_TABLE_ID)]
     ) 

```

ARP LOOKUP�ơ��֥�Ǥϡ�����ȥ���˥ѥ��åȤ�ɤΤ褦�˰�����
�䤤��碌�뤿��packet in�����������뤿�ἡ�Τ褦�ˤ���
�ե�����ȥ���ɲä��롥
```
    send_flow_mod_add(
      dpid,
      table_id: ARP_LOOKUP_TABLE_ID,
      idle_timeout: 0,
      priority: 1,
      match: Match.new,
      instructions: Apply.new(SendOutPort.new(:controller)),
    )
```

EGRESS�ơ��֥�ϡ��ǽ�Ū�˥ѥ��åȤ���������ơ��֥�ˤʤäƤ��롥
�������뤿��Υݡ��Ȥ�reg1�����򤵤�Ƥ���ΤǤ��ΰ��谸��
�������뤿�ᡤ���Τ褦�ˤ��ƥե�����ȥ���ɲä��롥
```
    send_flow_mod_add(
      dpid,
      table_id: EGRESS_TABLE_ID,
      idle_timeout: 0,
      priority: 0,
      match: Match.new,
      instructions: Apply.new(NiciraSendOutPort.new(:reg1)),
    )
```

�ʾ夬switch ready���ɲä��٤��ե�����ȥ�Ǥ��롥
�⤦1�ĺǽ���ɲä��٤��ե�����ȥ꤬���뤬�������
���μ¹Ի����ɲä����ե�����ȥ����ǽҤ٤롥

�¹Ի����ɲä����ե�����ȥ�ˤĤ��ƽҤ٤롥
packet in��ARP��ץ饤��IPv4�ѥ��åȤ��Ф��������롥
�롼�������ä�ARP�ꥯ�����Ȥ��Ф���ARP��ץ饤���֤äƤ���
�Ȥ�������MAC���ɥ쥹�����Ǥ����Τǡ�����ȥ����ί�ޤäƤ���
�ѥ��åȤ��������롥�ޤ�Ʊ���ˡ����夽�ΰ���IP���ɥ쥹(�ǽ�˽Ҥ٤���Ǥ���host2��IP���ɥ쥹)���Ф���ѥ��åȤϥ���ȥ�����䤤��碌�뤳�Ȥʤ�(packet in�򵯤�����)��
�������Ƥۤ����ΤǤ��Τ���Υե�����ȥ���ɲä��롥
�Ĥޤ�ARP��ץ饤��������MAC���ɥ쥹�ϥѥ��åȤ�������MAC���ɥ쥹�Ȥʤꡤ
������ݡ��Ȥ�ARP��ץ饤�ѥ��åȤ������ݡ��Ȥ򤽤Τޤ��Ѥ�����ɤ���
�����ƥѥ��åȤ��������뤿���EGRESS�ơ��֥�˰�ư���롥
��äƼ��Τ褦�ˤ��ƥե�����ȥ��ARP LOOKUP�ơ��֥���ɲä��롥
```
    action = [NiciraRegLoad.new(message.source_mac.to_i,:destination_mac_address),
               NiciraRegLoad.new(message.in_port, :reg1)]
    send_flow_mod_add(
      dpid,
      table_id: ARP_LOOKUP_TABLE_ID,
      idle_timeout: 0,
      priority: 2,
      match: Match.new(
     ether_type: ETHER_TYPE_IPv4,
     ipv4_destination_address: message.sender_protocol_address),
      instructions:[Apply.new(action),GotoTable.new(EGRESS_TABLE_ID)]
    )
```
�ʾ�μ�����host�֤�ping��������뤳�Ȥ��Ǥ�����
���������ΤޤޤǤϥ롼�����Ф���ping�������Ǥ��Ƥ��ʤ���
���ߤΤޤޤǥ롼�����Ф���ping�����ä��Ȥ���ͤ���ȡ�
Classifier�ơ��֥롤Routing�ơ��֥롤Interface lookup�ơ��֥��Ф�
ARP lookup�ơ��֥��packet in��ȯ������packet_in_ipv4���ƤӽФ���롥
�롼�����Ф���ping�ξ�硤�ʲ���packet_in_icmpv4_echo_request���ƤӽФ���롥
OpenFlow1.0�ǤΥ롼���Ǥ�arp̤���ξ�硤�ѥ��åȤ����������Ƥ��뤬��
�������ξ�����Ϥ����ѥ��åȤ�ʬ���äƤ���Τǡ����Τޤޥѥ��åȤ������֤��Ф褤�Τ�
packet_in_icmpv4_echo_request����Ǥ��Τޤ�send_packet_out������ɤ���


```
  def packet_in_ipv4(dpid, message)
    if forward?(message)
      forward(dpid, message)
    elsif message.ip_protocol == 1
      icmp = Icmp.read(message.raw_data)
      packet_in_icmpv4_echo_request(dpid, message) if icmp.icmp_type == 8
    else
      logger.debug "Dropping unsupported IPv4 packet: #{message.data}"
    end
  end

  # rubocop:disable MethodLength
  def packet_in_icmpv4_echo_request(dpid, message)
    icmp_request = Icmp.read(message.raw_data)
    interface = @interfaces.find_by(port_number: message.in_port)
      send_packet_out(dpid,
                      raw_data: create_icmp_reply(icmp_request).to_binary,
                      actions: SendOutPort.new(message.in_port))

    end
  end

```

�����������ǵ���Ĥ��ʤ���Фʤ�ʤ��Τ�������ޤǤΥե�����ȥ�Τޤޤ���
INTERFACE LOOKUP�ơ��֥���ɲä��줿�ʲ��Υե�����ȥ�ˤ�äơ�
��å�������������MAC���ɥ쥹���롼�����ȤΤ�Τ˽񤭴������Ƥ��ޤäƤ��뤿�ᡤ
�������Υۥ��Ȥ˥ѥ��åȤ������֤����Ȥ��Ǥ��ʤ���

```
    actions = [NiciraRegLoad.new(each.fetch(:port),:reg1),
               SetSourceMacAddress.new(each.fetch(:mac_address))]
    send_flow_mod_add(
       dpid,
       table_id: INTERFACE_LOOKUP_TABLE_ID,
       idle_timeout: 0,
       priority: 0,
       match: Match.new(
              reg0: ip_address_mask.to_i,
              reg0_mask: default_mask2.to_i),
       instructions: [Apply.new(actions),GotoTable.new(ARP_LOOKUP_TABLE_ID)]
     ) 
```

������ROUTING�ơ��֥�ǡ��⤷�롼�����Υѥ��åȤξ�硤INTERFACE LOOKUP�ơ��֥��
��ư������������ȥ����ľ���Ϥ����ɤ���
���Τ���switch_ready�Ǽ��Υե�����ȥ���ɲä��Ƥ�����

```
    send_flow_mod_add(
       dpid,
       table_id: ROUTING_TABLE_ID,
       idle_timeout: 0,
       priority: 0,
       match: Match.new(
              ether_type: ETHER_TYPE_IPv4,
              ipv4_destination_address: ip_address,
              dst_protocol_address:ip_address),
       instructions: [Apply.new(SendOutPort.new(:controller))]
     )     
```

#�¹Է��
�ޤ��¹Գ��ϻ��Υե�����ȥ�򼨤���
```
$ sudo ovs-ofctl dump-flows br0x1 --protocols=OpenFlow13
OFPST_FLOW reply (OF1.3) (xid=0x2):
 cookie=0x0, duration=8.040s, table=0, n_packets=0, n_bytes=0, priority=0,arp actions=goto_table:2
 cookie=0x0, duration=8.003s, table=0, n_packets=0, n_bytes=0, priority=0,ip actions=goto_table:3
 cookie=0x0, duration=8.003s, table=2, n_packets=0, n_bytes=0, priority=0,arp,in_port=1,arp_tpa=192.168.1.1,arp_op=1 actions=move:NXM_OF_ETH_SRC[]->NXM_OF_ETH_DST[],set_field:00:00:00:01:00:01->eth_src,set_field:2->arp_op,move:NXM_NX_ARP_SHA[]->NXM_NX_ARP_THA[],move:NXM_OF_ARP_SPA[]->NXM_OF_ARP_TPA[],set_field:00:00:00:01:00:01->arp_sha,set_field:192.168.1.1->arp_spa,load:0xffff->OXM_OF_IN_PORT[],load:0x1->NXM_NX_REG1[],goto_table:6
 cookie=0x0, duration=8.003s, table=2, n_packets=0, n_bytes=0, priority=0,arp,in_port=1,arp_tpa=192.168.1.1,arp_op=2 actions=CONTROLLER:65535
 cookie=0x0, duration=7.998s, table=2, n_packets=0, n_bytes=0, priority=0,arp,in_port=2,arp_tpa=192.168.2.1,arp_op=1 actions=move:NXM_OF_ETH_SRC[]->NXM_OF_ETH_DST[],set_field:00:00:00:01:00:02->eth_src,set_field:2->arp_op,move:NXM_NX_ARP_SHA[]->NXM_NX_ARP_THA[],move:NXM_OF_ARP_SPA[]->NXM_OF_ARP_TPA[],set_field:00:00:00:01:00:02->arp_sha,set_field:192.168.2.1->arp_spa,load:0xffff->OXM_OF_IN_PORT[],load:0x2->NXM_NX_REG1[],goto_table:6
 cookie=0x0, duration=7.994s, table=2, n_packets=0, n_bytes=0, priority=0,arp,in_port=2,arp_tpa=192.168.2.1,arp_op=2 actions=CONTROLLER:65535
 cookie=0x0, duration=8.003s, table=2, n_packets=0, n_bytes=0, priority=0,arp,reg1=0x1 actions=set_field:00:00:00:01:00:01->eth_src,set_field:00:00:00:01:00:01->arp_sha,set_field:192.168.1.1->arp_spa,goto_table:6
 cookie=0x0, duration=7.988s, table=2, n_packets=0, n_bytes=0, priority=0,arp,reg1=0x2 actions=set_field:00:00:00:01:00:02->eth_src,set_field:00:00:00:01:00:02->arp_sha,set_field:192.168.2.1->arp_spa,goto_table:6
 cookie=0x0, duration=7.977s, table=3, n_packets=0, n_bytes=0, priority=40024,ip,nw_dst=192.168.1.1 actions=CONTROLLER:65535
 cookie=0x0, duration=7.971s, table=3, n_packets=0, n_bytes=0, priority=40024,ip,nw_dst=192.168.2.1 actions=CONTROLLER:65535
 cookie=0x0, duration=7.981s, table=3, n_packets=0, n_bytes=0, priority=10,ip,nw_dst=192.168.1.0/24 actions=move:NXM_OF_IP_DST[]->NXM_NX_REG0[],goto_table:4
 cookie=0x0, duration=7.974s, table=3, n_packets=0, n_bytes=0, priority=10,ip,nw_dst=192.168.2.0/24 actions=move:NXM_OF_IP_DST[]->NXM_NX_REG0[],goto_table:4
 cookie=0x0, duration=7.967s, table=4, n_packets=0, n_bytes=0, priority=0,reg0=0xc0a80100/0xffffff00 actions=load:0x1->NXM_NX_REG1[],set_field:00:00:00:01:00:01->eth_src,goto_table:5
 cookie=0x0, duration=7.964s, table=4, n_packets=0, n_bytes=0, priority=0,reg0=0xc0a80200/0xffffff00 actions=load:0x2->NXM_NX_REG1[],set_field:00:00:00:01:00:02->eth_src,goto_table:5
 cookie=0x0, duration=7.961s, table=5, n_packets=0, n_bytes=0, priority=1 actions=CONTROLLER:65535
 cookie=0x0, duration=7.959s, table=6, n_packets=0, n_bytes=0, priority=0 actions=output:NXM_NX_REG1[]
```
�ץ����������Ǽ�������switch_ready���ɲä��٤��ե�����ȥ꤬�ɲ�
����Ƥ��뤳�Ȥ���ǧ�Ǥ��롥
�ޤ�host1����host2��ping�����롥
```
$ ./bin/trema netns host1
root@ensyuu2-VirtualBox:~/class/simple_router-team_alpha# ping 192.168.2.2
PING 192.168.2.2 (192.168.2.2) 56(84) bytes of data.
64 bytes from 192.168.2.2: icmp_seq=1 ttl=64 time=100 ms
64 bytes from 192.168.2.2: icmp_seq=2 ttl=64 time=0.490 ms
64 bytes from 192.168.2.2: icmp_seq=3 ttl=64 time=0.064 ms
64 bytes from 192.168.2.2: icmp_seq=4 ttl=64 time=0.043 ms
64 bytes from 192.168.2.2: icmp_seq=5 ttl=64 time=0.060 ms
64 bytes from 192.168.2.2: icmp_seq=6 ttl=64 time=0.071 ms
64 bytes from 192.168.2.2: icmp_seq=7 ttl=64 time=0.061 ms
64 bytes from 192.168.2.2: icmp_seq=8 ttl=64 time=0.065 ms
```
������ping���Ϥ��Ƥ��뤳�Ȥ�ʬ���ä���³���Ƶդ�host2����host1�ˤ�
ping���������롥
```
$ ./bin/trema netns host2
root@ensyuu2-VirtualBox:~/class/simple_router-team_alpha# ping 192.168.1.2
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=64 time=0.247 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=64 time=0.047 ms
64 bytes from 192.168.1.2: icmp_seq=3 ttl=64 time=0.045 ms
64 bytes from 192.168.1.2: icmp_seq=4 ttl=64 time=0.067 ms
64 bytes from 192.168.1.2: icmp_seq=5 ttl=64 time=0.051 ms
64 bytes from 192.168.1.2: icmp_seq=6 ttl=64 time=0.052 ms
```
��������������Ϥ��Ƥ��뤳�Ȥ�ʬ���ä���������ARP LOOKUP�ơ��֥�Υե��ơ��֥��
ARP̤�����ä��ѥ��åȤ򺣸��Ʊ��������Ϥ���٤Υե�����ȥ꤬�ɲä���Ƥ��뤫
��ǧ���롥
```
$ sudo ovs-ofctl dump-flows br0x1 --protocols=OpenFlow13
OFPST_FLOW reply (OF1.3) (xid=0x2):
-(��ά)-
cookie=0x0, duration=179.715s, table=5, n_packets=13, n_bytes=1274, priority=2,ip,nw_dst=192.168.2.2 actions=load:0xfe1860a1cc0a->NXM_OF_ETH_DST[],load:0x2->NXM_NX_REG1[],goto_table:6
 cookie=0x0, duration=179.664s, table=5, n_packets=13, n_bytes=1274, priority=2,ip,nw_dst=192.168.1.2 actions=load:0x67e908b719e->NXM_OF_ETH_DST[],load:0x1->NXM_NX_REG1[],goto_table:6
 -(��ά)-
```
�������ե�����ȥ꤬�ɲä���Ƥ��뤳�Ȥ���ǧ�Ǥ�����
�Ǹ�˥ۥ��Ȥ���롼���ؤ�ping���Ϥ������ǧ���롥
```
$ ./bin/trema netns host1
root@ensyuu2-VirtualBox:~/class/simple_router-team_alpha# ping 192.168.1.1
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
64 bytes from 192.168.1.1: icmp_seq=1 ttl=128 time=5.88 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=128 time=7.79 ms
64 bytes from 192.168.1.1: icmp_seq=3 ttl=128 time=7.35 ms
64 bytes from 192.168.1.1: icmp_seq=4 ttl=128 time=6.45 ms
64 bytes from 192.168.1.1: icmp_seq=5 ttl=128 time=14.5 ms
```

�ʾ�Τ褦�˥롼���ؤ�ping���������Ϥ��Ƥ��뤳�Ȥ�ʬ���ä���
�����η�̤����׵ᤵ�줿���ͤ�OpenFlow1.3�ǥ롼���μ������Ǥ��Ƥ��뤳�Ȥ�
��ǧ�Ǥ�����
