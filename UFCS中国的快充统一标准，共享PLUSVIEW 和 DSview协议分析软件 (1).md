

@[TOC](文章目录)

---

# 前言

		

---
不同品牌终端和适配器之间不能有效识别，只能实现较低功率的充电。一方面，用户快充体验受到很大的制约和限制，不兼容问题成为用户的一大痛点；另一方面，由于充电标准不统一，导致产业链上下游厂商研发通用快充电源芯片和配件的风险和成本相对高昂。技术制式的不统一也将妨碍终端绿色能源和循环经济的长期发展。

目前USB-PD已经成为欧洲的标准，所有厂商都表示支持。中国也在2021年提出了UFCS标准，目前正在积极推广中，基于中国是世界第一大手机市场，更是第一大的电子产品制造国，UFCS应该有着巨大的前景！


# 一、UFCS 与USB-PD的优缺点比较
UFCS 的物理层就是UART串口，几乎所有的MCU都自带UART串口硬件，这就让产品开发起来很简单了。而USB-PD 采用BMC编码，简单说就是用脉冲的宽窄代表1和0，而且信号的高电平才1V左右，这就必须要有硬件比较器，增加了成本。好处当然是抗干扰的能力更强。然而UART经过几十年的发展，也有许多的工业应用，比如RS232， RS485， LIN， MODBUS......, 足够应付各种场景了吧？！

USB-PD的频率是300Kbps， 软件发送边沿的跳变就要600KHz, 接收采样就需要5-10倍的速度，才能有效保证不丢数据位，这对于ARM32位的MCU也需要主频48MHz 才能够用。对于应用量更大的8位MCU，就只能靠硬件来实现物理层了，纯软件处理速度跟不上。用硬件的缺点首先是增加了成本，其次硬件开发周期长，合并进入芯片，这个过程至少也要半年。芯片debug也就是流片的成本，又要几十万，上百万。而在UART基础上做个UFCS的协议软件，也就2-3周时间。

还有一点很重要，USB-PD需要额外的两条CC线，来传送数据，完成通讯。UFCS则是利用USB原有的D+，D-两条数据线来通讯。这和高通，华为等手机大厂的方式一样，好处又是硬件简单。而且可以在USB-A/B USB-C 等所有USB 口上实现，USB-PD就只支持新的type-C接口了。

基于以上的点看，USB-PD标准发布至今已经10年了，才刚刚开始大面积应用，应该是跟各种硬件成本和应用难度不无关系，重新开发硬件，开发软件都不是简单的事情。

当然，UFCS目前功能还比较简单。USB-PD的以下功能，UFCS暂时还没有：
1. Dual-role, 就是一个口，又能充电也能放电，就是有的手机口还能当充电宝，给其他手机充电。只看手机对手机，感觉比较鸡肋。但是，如果发挥想象力，你的手机，电脑，显示器，笔电等等电器都用USB线连在一起，只需要一个充电器了。乱七八糟的插座，是不是就不要那么多了？
2. Alternative, 就是这个PD口可以代替视频线，音频线，HDMI，雷电，等等，你的显示器，只需要一根线跟主机连接，连电源线也不要了。所有的设备之间连接只有USB，不管是信号线还是电源线，只有USB一种，简单了吧？

以上是最近几个月开发USB-PD和UFCS，的一些体会，下面也分享一些有用的东东给大家，希望对UFCS的发展添砖加瓦。（本文欢迎转发，但请保留作者联系信息 i2tv@qq.com）

# 二、UFCS协议分析工具
## 1.PLUSVIEW 
PLUSVIEW 是[sigrok](https://sigrok.org/wiki/Main_Page)开源的协议分析软件，无数的协议分析仪和示波器都用了这家的软件，上百种协议的软件都是开源的，比如UART, IIC, SPI, USB-PD, LIN, CAN呀， 相当赞！我来贡献UFCS协议分析包，不知道能否被接受呢。先共享出来给大家用吧。
![UFCS协议](https://img-blog.csdnimg.cn/255b230a77f841938a898e0e8c04466f.png#pic_center)
![UFCS协议](https://img-blog.csdnimg.cn/fb56dde1108c4f799be719f47121a941.png#pic_center)
硬件价格从20元人民币到几万美元的都有，分析UFCS就去某宝或者PDD上买个几十元的就够用了。协议支持都是基于[sigrok](https://sigrok.org/wiki/Main_Page)开源共享的，只是采样速度和阈值电平的调节不同。以上的图是用[muselab](https://item.taobao.com/item.htm?spm=a230r.1.14.16.a61e296aiUorS1&id=642354490902&ns=1&abbucket=3#detail)家的板子采的，不是最低价，主要表示对开源的支持！

说到开源，我觉得是一种共享知识的模式，能让软件硬件迅速发展和普及，建议UFCS等等协议也应该开源，协议开放，并且提供参考源代码，参考硬件设计，这样发展就快了。第一步我先把自己写的UFCS协议分析包开源提供在这里，或者Gitbee, github。

## 2.DSview
另外，还要分析USB-PD的话，推荐[DSlogic](https://dreamsourcelab.cn/product/dslogic-series/)。也是基于sigrok的共享软件，二次开发的。
![UFCS协议分析](https://img-blog.csdnimg.cn/aa68447270f64b0797f12b47449fd375.png#pic_center)
![UFCS协议分析](https://img-blog.csdnimg.cn/313d3a682b694c44b0b3749a10a67b42.png#pic_center)
[DSlogic](https://dreamsourcelab.cn/product/dslogic-series/)主要优点是阈值电平可以调节，也就是识别更低的电平，因为USB-PD的高电平1V左右，普通MCU无法直接识别，必须有硬件比较器的电平转换。
另外一个重要优点就是上图蓝色的部分，直接点击搜索需要的特征值，然后点击就可以迅速找到数据包的位置，这个对于协议分析的效率很高，pulseview 要左右拉来拉去，用肉眼找数据，还是很费脑子的；）

个人版300-500元的价格，跟老外的价格比较起来，就是挣个茶水费喽。

## 3.UFCS协议分析软件插件共享
基于sigrok的协议分析软件，都有一个decoders目录，里面每个子目录是一个协议，包括两个文件__init__.py 和 pd.py。所以看这些python源码就可以知道上百种协议的原理了。改到MCU上，用c语言实现也就轻松愉快。
[下载链接1](https://gitee.com/renxn/ufcs.git)
https://gitee.com/renxn/ufcs.git
[下载链接2](https://github.com/392625227/UFCS-protocol-compare-with-USB-PD)

https://github.com/392625227/UFCS-protocol-compare-with-USB-PD
主要几个文件，最重要的最长，放在文章最后面。.sr 文件是pulseview软件抓取的波形，先选UART，然后再选UFCS，就可以分析UFCS数据包。.dsl是DSview软件抓的波形。

1__init__.py 主要就一句话，为了plusview， DSview软件找到这个协议
```python
from .pd import Decoder
```
2. pd.py 有点儿长，凑合看吧；）
```python
##
## This file is part of the libsigrokdecode project.
##
## Copyright (C) 2023 edison ren <i2tv@qq.com>
## ref from uart & usb_power_delivery 
##
## This program is free software; you can redistribute it and/or modify
## it under the terms of the GNU General Public License as published by
## the Free Software Foundation; either version 2 of the License, or
## (at your option) any later version.
##
## This program is distributed in the hope that it will be useful,
## but WITHOUT ANY WARRANTY; without even the implied warranty of
## MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
## GNU General Public License for more details.
##
## You should have received a copy of the GNU General Public License
## along with this program; if not, see <http://www.gnu.org/licenses/>.
##

import sigrokdecode as srd

# Control Message type
CTRL_TYPES = {
    0: 'PING',
    1: 'ACK',
    2: 'NCK',
    3: 'ACCEPT',
    4: 'SOFT RESET',
    5: 'POWER READY',
    6: 'GET OUTPUT CAP',
    7: 'GET SOURCE INFO',
    8: 'GET SINK INFO',
    9: 'GET CABLE INFO',
    10: 'GET DEVICE INFO',
    11: 'GET ERROR INFO',
    12: 'DETECT CABLE INFO',
    13: 'START CABLE DETECT',
    14: 'END CABLE DETECT',
    15: 'EXIT UFCS MODE',
}

# Data message type
DATA_TYPES = {
    1: 'OUTPUT CAP',
    2: 'REQUEST',
    3: 'SOURCE INFO',
    4: 'SINK INFO',
    5: 'CABLE INFO',
    6: 'DEVICE INFO',
    7: 'ERROR INFO',
    8: 'CONFIG WATCHDOG',
    9: 'REFUSE',
    10: 'Verify_Request',
    11: 'Verify_Response',
    255: 'Test Request'
}

class Decoder(srd.Decoder):
    api_version = 3
    id = 'ufcs'
    name = 'UFCS'
    longname = 'Universal Fast Charging Specification'
    desc = 'Universal fast charging specification for mobile devices. T/TAF 083-2021. Coding by edison ren 2023.1.25 <i2tv@qq.com>'
    license = 'gplv2+'
    inputs = ['uart']
    outputs = []
    tags = ['PC/Mobile']

    options = (
        {'id': 'fulltext', 'desc': 'Full text decoding of packets',
         'default': 'no', 'values': ('yes', 'no')},
    )
    annotations = (
        ('type', 'Packet Type'),
        ('training', 'Training'),
        ('header', 'Header'),
        ('data', 'Data'),
        ('crc', 'Checksum'),
        ('warnings', 'Warnings'),
        ('src', 'Source Message'),
        ('snk', 'Sink Message'),
        ('payload', 'Payload'),
        ('text', 'Plain text'),
        ('cable', 'Cable Message'),
        ('reserved', 'Reserved'),
        
    )
    annotation_rows = (
       ('phase', 'Parts', (1, 2, 3, 4,)),
       ('payload', 'Payload', (8,)),
       ('type', 'Type', (0, 6, 7, 10, 11)),
       ('warnings', 'Warnings', (5,)),
       ('text', 'Full text', (9,)),
    )

    def __init__(self):
        self.reset()

    def reset(self):
        self.ss_block = None    # start pos with samles for show
        self.es_block = None    # end pos with samples for show
        self.dataidx = 0        # data pos with u8 in packet 
        self.datapkt = []       # data buffer for packet, 4:128
        self.head = []          # header 2*u8 
        self.bytepos = []       # (s, e) pos of byte (start, end) 
        self.text = ''          # info string 

    def start(self):
        self.out_ann = self.register(srd.OUTPUT_ANN)

    def head_id(self):
        return (self.head[0] >> 1) & 15

    def head_power_role(self):
        return (self.head[0] >> 5) & 7

    def head_rev(self):
        return (self.head[1] >> 3) & 31

    def head_type(self):
        return self.head[2]

    def data_len(self):
	# data_msg count = data[3], ctrl_msg count = 0 
        return self.head[3] if (self.head[1] & 7) == 1 else 0

    def putx(self, s0, s1, data):
        self.put(s0, s1, self.out_ann, data)

    def putwarn(self, longm, shortm):
        self.putx(0, -1, [8, [longm, shortm]])

    def compute_crc8(self):
        CRC_8_POLYNOMIAL = 0x29
        rCRC = 0

        for s in range(self.plen - 1):
            rCRC ^= self.datapkt[s]
            for i in range(8):
                if (rCRC & 0x80):
                    rCRC = (rCRC << 1) ^ CRC_8_POLYNOMIAL
                else:
                    rCRC = (rCRC << 1)

        return (rCRC & 0xff)

    def get_short(self, i):
        k = [self.datapkt[i], self.datapkt[i+1]]
        val = k[0] <<8 | (k[1])
        return val

    def get_word(self, i):
        hi = self.get_short(i)
        lo = self.get_short(i+2)
        return lo | (hi << 16)
    
    def get_dword(self, i):
        hi = self.get_word(i)
        lo = self.get_word(i+4)
        return lo | (hi << 32)
    
    def puthead(self):
        # message reviever, SRC, SNK, Cable
        pwr_role = self.head_power_role()
        if pwr_role == 1:
            ann_type = 6
            role = 'SRC'
        elif pwr_role == 2:
            ann_type = 7
            role = 'SNK'
        elif pwr_role == 3:
            ann_type = 10
            role = 'Cable'
        else:
            ann_type = 11
            role = 'Reserved'

        t = self.head_type()
        
        if self.data_len() == 0:
            if t > 15:
                shortm = 'reserved cmd'
            else:
                shortm = CTRL_TYPES[t]
        else:
            shortm = DATA_TYPES[t] if t in DATA_TYPES else 'DAT???'

        longm = '(r{:d}) {:s}[{:d}]: {:s}'.format(self.head_rev(), role, self.head_id(), shortm)
        self.putx(self.bytepos[0][0], self.bytepos[2][1], [ann_type, [longm, shortm]])
        self.text += longm

    def get_source_cap(self):
        numbpdo = int(self.data_len()) // 8
        strpdo = ''

        for numb in range(numbpdo):
            pdo = self.get_dword(4 + 8 * numb)
            mode = (pdo >> 60) & 15
            step_ma = ((pdo >> 57) & 7) * 10 + 10
            step_mv = ((pdo >> 56) & 1) * 10 + 10
            max_mv = ((pdo >> 40) & 0xffff) * 0.01
            min_mv = ((pdo >> 24) & 0xffff) * 0.01
            max_ma = ((pdo >> 8) & 0xffff) * 0.01
            min_ma = (pdo & 0xff) * 0.01
            strpdo = '[%d] %g/%gV *%gmv %g/%gA *%gma ' \
                    % (mode, min_mv, max_mv, step_mv, min_ma, max_ma, step_ma)

            s0 = self.bytepos[4 + 8 * numb][0]
            s1 = self.bytepos[4 + 8 * numb + 7][1]
            # show data in hex
            self.putx(s0, s1, [3, ['[%d]%08x' % (numb, pdo), 'D%d' % (numb)]])
            # show data in comment
            txt = '(PDO %s)' % strpdo
            self.putx(s0, s1, [8, [txt, txt]])
            # show data in full text string
            self.text += ' - ' + txt

        return self.text

    def get_request(self):
        rdo = self.get_dword(4)
        
        mode = (rdo >> 60) & 15

        curr = (rdo & 0xffff) * 0.01
        volt = ((rdo >> 16) & 0xffff) * 0.01

        s = '%gV %gA' % (volt, curr)

        return rdo, '(PDO #%d) %s' % (mode, s)
       
    def get_src_info(self):
        d = self.get_dword(4)
        #self.putwarn('bbbbbb%08x' % self.get_dword(4), 'aaa')

        it = ((d >> 40 ) & 0xff) - 50
        its = ',internal temp %s' % ('%dC' % it if it > -50 else 'no data')
        
        pt = ((d >> 32 ) & 0xff) - 50
        pts = ',usb port temp %s' % ('%dC' % pt if pt > -50 else 'no data')
        
        curr = (d & 0xffff) * 0.01
        volt = ((d >> 16) & 0xffff) * 0.01

        return d, '(SRC info: %gV %gA %s %s)' % (volt, curr, its, pts)

    def get_snk_info(self):
        d = self.get_dword(4)

        it = ((d >> 40 ) & 0xff) - 50
        its = ',battery temp %s' % ('%dC' % it if it > -50 else 'no data')
        
        pt = ((d >> 32 ) & 0xff) - 50
        pts = ',usb port temp %s' % ('%dC' % pt if pt > -50 else 'no data')
        
        curr = (d & 0xffff) * 0.01
        volt = ((d >> 16) & 0xffff) * 0.01

        return d, '(SNK info: %gV %gA %s %s)' % (volt, curr, its, pts)

    def get_cable_info(self):
        d = self.get_dword(4)

        id = ((d >> 48 ) & 0xffff)
        ids = 'Cable VID %04X' % (id)
        
        emk = ((d >> 32 ) & 0xffff)
        emks = 'Emark VID %04X' % emk
        
        imp = ((d >> 16) & 0xffff)
        imps = 'IMP %d' % imp 

        curr = (d & 0xff)
        volt = ((d >> 8) & 0xff)

        return d, '(%s %s %s %gV %gA)' % (ids, emks, imps, volt, curr)

    def get_device_info(self):
        d = self.get_dword(4)

        id = ((d >> 48 ) & 0xffff)
        ids = 'Device VID %04X' % (id)
        
        pid = ((d >> 32 ) & 0xffff)
        pids = 'Protocol IC VID %04X' % emk
        
        hwv = ((d >> 16) & 0xffff)
        hwvs = 'HW rev%04X' % imp 

        swv = (d & 0xffff) 
        swvs = 'SW rev%04X' % imp 

        return d, '(%s %s %s %s)' % (ids, pids, hwv, swv)

    def get_error_info(self):
        d = self.get_word(4)

        id = ((d >> 15 ) & 0x1ffff)
        ids = 'Reversed %04X' % (id)
        
        e14 = ((d >> 14 ) & 1)
        e14s = 'Watchdog %s' % ('OK' if e14 == 0 else 'NG!')
        
        e13 = ((d >> 13 ) & 1)
        e13s = 'CRC %s' % ('OK' if e13 == 0 else 'NG!')
        
        e12 = ((d >> 12 ) & 1)
        e12s = 'Input %s' % ('OK' if e12 == 0 else 'NG!')
        
        e11 = ((d >> 11 ) & 1)
        e11s = 'Current loss %s' % ('OK' if e11 == 0 else 'NG!')
        
        e10 = ((d >> 10 ) & 1)
        e10s = 'UVP %s' % ('OK' if e10 == 0 else 'NG!')
        
        e9 = ((d >> 9 ) & 1)
        e9s = 'OVP %s' % ('OK' if e9 == 0 else 'NG')
        
        e8 = ((d >> 8 ) & 1)
        e8s = 'D+OVP %s' % ('OK' if e8 == 0 else 'NG')
        
        e7 = ((d >> 7 ) & 1)
        e7s = 'D-OVP %s' % ('OK' if e7 == 0 else 'NG')
        
        e6 = ((d >> 6 ) & 1)
        e6s = 'CCOVP %s' % ('OK' if e6 == 0 else 'NG')
        
        e5 = ((d >> 5 ) & 1)
        e5s = 'Battery OTP %s' % ('OK' if e5 == 0 else 'NG')
        
        e4 = ((d >> 4 ) & 1)
        e4s = 'USB port OTP %s' % ('OK' if e4 == 0 else 'NG')
        
        e3 = ((d >> 3 ) & 1)
        e3s = 'SCP %s' % ('OK' if e3 == 0 else 'NG')
        
        e2 = ((d >> 2 ) & 1)
        e2s = 'OCP %s' % ('OK' if e2 == 0 else 'NG')
        
        e1 = ((d >> 1 ) & 1)
        e1s = 'UVP %s' % ('OK' if e1 == 0 else 'NG')
        
        e0 = ((d >> 0 ) & 1)
        e0s = 'OVP %s' % ('OK' if e0 == 0 else 'NG')
        
        return d, '(%s %s %s %s %s %s %s %s %s %s %s %s %s %s %s %s)' % (ids, \
                e14, e13, e12, e11, e10, e9, e8, e7, e6, e5, e4, e3, e2, e1, e0)

    def config_watchdog(self):
        d = self.get_short(4)
        
        return d, 'Watchdog overflow time %dms' % d
    
    def refuse(self):
        d = self.get_word(4)

        rev = ((d >> 28 ) & 0xf)
        revs = ('Reversed %01X' % rev) if rev else ''
        rev2 = ((d >> 19 ) & 0xf)
        revs2 = ('Reversed %01X' % rev2) if rev2 else ''
                
        id = ((d >> 24 ) & 0xf)
        ids = 'MSG #%d' % (id)
        
        t = ((d >> 16 ) & 7)
        ts = ', Type %d' % ts
        
        c = ((d >> 8) & 0xff)
        cs = ', CMD %d' % c

        r1 = ((d >> 0) & 0x7)
        rs1  = ', unknow'      if r1 == 1 else ''
        rs1 += ', not support' if r1 == 2 else ''
        rs1 += ', busy'        if r1 == 3 else ''
        rs1 += ', over range'  if r1 == 4 else ''
        rs1 += ', other'       if r1 == 5 else ''

        return d, 'Refuse why? $s' % (ids + ts + cs + revs +revs2 + rs1)

    def verify_request(self):
        id = self.datapkt[4]
        dhi = self.get_dword(5)
        dlo = self.get_dword(5 + 8) 
        
        return id, 'Verify request key ID %d, random %08x%08x' % (id, dhi, dhl)
        
    def verify_response(self):
        key = self.get_dword(4)
        rand = self.get_word(4+8) 
        
        return id, 'Verify request data %08x, random %04x' % (key, rand)
            
    def decode_data_msg(self):
        dlen = self.data_len()

        # no data, do nothing
        if dlen == 0:
            return

        txt = ''
        t = self.head_type()
                            
        # SRC CAP
        if   t == 1:
            self.get_source_cap()
            # src cap has n(1-15) PDO, putx will be run n times within function,
            # so difference here
            return 
           
        # request
        elif t == 2:
            d, txt = self.get_request()
            
        
        # SRC info
        elif t == 3:
            d, txt = self.get_src_info()

        # SNK info
        elif t == 4:
            d, txt = self.get_snk_info()
            
        # cable info
        elif t == 5:
            d, txt = self.get_cable_info()

        # device info
        elif t == 6:
            d, txt = self.get_device_info()

        # error info
        elif t == 7:
            d, txt = self.get_error_info()

        # config watchdog
        elif t == 8:
            d, txt = self.config_watchdog()

        # refuse
        elif t == 9:
            d, txt = self.refuse()

        # verify request
        elif t == 10:
            d, txt = self.verify_request()

        # verify response
        elif t == 11:
            d, txt = self.verify_response() 

        # test request
        else:
            d, txt = 0xffffffff, 'TODO...' 
        
        s0 = self.bytepos[4][0]
        s1 = self.bytepos[dlen+3][1]
        # show data in hex
        self.putx(s0, s1, [3, ['H:%08x' % d, 'DATA']])
        # show data in comment
        self.putx(s0, s1, [8, [txt, txt]])
        # show data in full text string
        self.text += ' - ' + txt
                
    def decode_pkt(self):
	# Packet header
        self.head = self.datapkt[0:4]
        
        self.putx(self.bytepos[0][0], self.bytepos[1][1],
                  [2, ['HEAD:%04x' % ((self.head[0] << 8) | self.head[1]), 'HD']])
        self.puthead()
        
	# Cmd
        self.putx(self.bytepos[2][0], self.bytepos[2][1],
                  [2, ['CMD:%02x' % (self.head[2]), 'CMD']])

	# Data length
        if self.data_len():
            self.putx(self.bytepos[3][0], self.bytepos[3][1], 
                      [2, ['LEN:%02x' % (self.head[3]), 'LEN']])
	
        # Decode data payload
        self.decode_data_msg()

        # CRC check 
        self.crc = self.datapkt[self.plen-1]
        ccrc = self.compute_crc8()
        if  self.crc != ccrc:
            self.putwarn('Bad CRC %02x != %02x' % (self.crc, ccrc), 'CRC!')
    
        self.putx(self.ss_block, self.es_block,
                      [4, ['CRC:%02x' % (self.crc), 'CRC']])
        
	# Full text trace
        if self.options['fulltext'] == 'yes':
            self.putx(self.bytepos[0][0], self.es_block, [9, [self.text, '...']])

    def decode(self, ss, es, data):
        ptype, rxtx, pdata = data
        self.ss_block, self.es_block = ss, es

        # just keep DATA
        if ptype != 'DATA':
            return

        # '0xAA' is the SOP of a ufcs package
        if pdata[0] == 0xaa:
            self.putx(ss, es, [3, ['SOP:0xaa', 'SOP']])
            self.reset()
            return

        # append data to packet
        self.datapkt.append(pdata[0])
        # append byte pos (start byte, end byte )
        self.bytepos.append((ss, es))
        			
    	# ctrl_msg or data_msg?
        if self.dataidx == 3 :
            if (self.datapkt[1] & 1) == 1:
                self.plen = self.datapkt[3] + 5
            else:
                self.plen = 4

        self.dataidx += 1

        # packet is full
        if self.dataidx == self.plen:
            self.decode_pkt()


```

---

# 总结
总的来说，UFCS简单实用，硬件用UART，软件开发容易，扩充功能方便，应该很有前景。个人对UFCS的发展壮大也贡献一点微薄之力。
