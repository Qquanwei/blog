---
layout: post
title: why xstream
image: xstream.jpeg
---

���ҽ�RxJS��������bundle�е�ʱ�����������200+Kb, ���ڲ��Ǻܴ��Ӧ����˵����ȷʵ�Ѿ������ˡ�Ȼ���ҾͰ�Ŀ��ת�Ƶ�[xstream](https://github.com/staltz/xstream)���ˡ�

xstream�ô�:

1. �����operator:

rxjs5������ʱ���Ѿ�����175+��operator, ���ݶ���ԭ���д󲿷��������ò����ģ�����`Andr�� Staltz`(xstream����) ����һ���ʾ���鷢�ִ�Լ��26��poerator�ǳ��õ�, ����`map`, `filter`,`combine`,`take`��, ��Щ���õ�operator�����ɵ���xstreamĬ��operator��, �Ӷ������˴���operator������������ �����Ҫ�����operator������`xstream/extra`���ҵ�.

2. ���ӷ���ֱ��������

RxJS ��Rx.NET��һ����еģ���������������ͳһ�ģ�������Щ��������֮ǰû�нӴ���Rx�ĳ�ѧ�߲������ر��Ѻã��������߶�������ʷ����ֱ�Ӹ�xstream����һ���µ����֣���ʵ��ʹ���в�û�����̫����鷳��������Ѿ��˽���rxjs ���Զ�һ�������ߵĲ��첿��: https://github.com/staltz/xstream/wiki/Migrating-from-RxJS

3. ���С

��㲻�ù�������ˣ�(rxjs)200+kb -> (xstream)30+kb, �����Ŀ��������Ƚ������⽫��һ���ܴ�����ơ�


TL;DR:

xstream��������Ϊ��cycle.js����Ҫһ�ָ�����������Ӧʽ���, �Լ���������ʹ�úͶ����ָ��Ѻ�, ��Զ���xstream���ֱȽ�����(����rxjs��combineLatest��rename��combine), ����ì�ܵ���xstream���ĵ��������в��㡣
