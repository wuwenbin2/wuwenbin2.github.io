---
layout: post
title: "Netty Pipeline Test"
description: "Netty Pipeline Test"
catagory: tech
tags: netty
---

#Server

```java:
package ChannelPipelineTest;

import java.net.InetSocketAddress;
import java.util.concurrent.Executors;

import org.jboss.netty.bootstrap.ServerBootstrap;
import org.jboss.netty.channel.Channel;
import org.jboss.netty.channel.ChannelEvent;
import org.jboss.netty.channel.ChannelHandlerContext;
import org.jboss.netty.channel.ChannelPipeline;
import org.jboss.netty.channel.ChannelPipelineFactory;
import org.jboss.netty.channel.Channels;
import org.jboss.netty.channel.ExceptionEvent;
import org.jboss.netty.channel.MessageEvent;
import org.jboss.netty.channel.SimpleChannelDownstreamHandler;
import org.jboss.netty.channel.SimpleChannelUpstreamHandler;
import org.jboss.netty.channel.socket.nio.NioServerSocketChannelFactory;

public class Server {
    public static void main(String args[]) {
        ServerBootstrap bootsrap = new ServerBootstrap(new NioServerSocketChannelFactory(Executors
                .newCachedThreadPool(), Executors.newCachedThreadPool()));
        bootsrap.setPipelineFactory(new PipelineFactoryTest());
        bootsrap.bind(new InetSocketAddress(8887));
    }

}

class PipelineFactoryTest implements ChannelPipelineFactory {

    @Override
    public ChannelPipeline getPipeline() throws Exception {
        ChannelPipeline pipeline = Channels.pipeline();
        pipeline.addLast("1", new UpstreamHandlerA());
        pipeline.addLast("2", new UpstreamHandlerB());
        pipeline.addLast("3", new DownstreamHandlerA());
        pipeline.addLast("4", new DownstreamHandlerB());
        pipeline.addLast("5", new UpstreamHandlerX());
        return pipeline;
    }
}

class UpstreamHandlerA extends SimpleChannelUpstreamHandler {
    @Override
    public void messageReceived(ChannelHandlerContext ctx, MessageEvent e)
            throws Exception {
        Channel ctxchannel = ctx.getChannel();
        Channel echannel = e.getChannel();

        System.out.println(ctxchannel.equals(echannel));// handle和event共享一个channel
        System.out
                .println("UpstreamHandlerA.messageReceived:" + e.getMessage().toString());
        ctx.sendUpstream(e);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, ExceptionEvent e) {
        System.out.println("UpstreamHandlerA.exceptionCaught:" + e.toString());
        e.getChannel().close();
    }
}

class UpstreamHandlerB extends SimpleChannelUpstreamHandler {
    @Override
    public void messageReceived(ChannelHandlerContext ctx, MessageEvent e)
            throws Exception {
        System.out
                .println("UpstreamHandlerB.messageReceived：" + e.getMessage().toString());
        ctx.sendUpstream(e);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, ExceptionEvent e) {
        System.out.println("UpstreamHandlerB.exceptionCaught：" + e.toString());
        e.getChannel().close();
    }
}

class UpstreamHandlerX extends SimpleChannelUpstreamHandler {
    @Override
    public void messageReceived(ChannelHandlerContext ctx, MessageEvent e)
            throws Exception {
        System.out
                .println("UpstreamHandlerX.messageReceived:" + e.getMessage().toString());
        e.getChannel().write(e.getMessage());
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, ExceptionEvent e) {
        System.out.println("UpstreamHandlerX.exceptionCaught");
        e.getChannel().close();
    }
}

class DownstreamHandlerA extends SimpleChannelDownstreamHandler {
    public void handleDownstream(ChannelHandlerContext ctx, ChannelEvent e)
            throws Exception {
        System.out.println("DownstreamHandlerA.handleDownstream");
        super.handleDownstream(ctx, e);
    }
}

class DownstreamHandlerB extends SimpleChannelDownstreamHandler {
    public void handleDownstream(ChannelHandlerContext ctx, ChannelEvent e)
            throws Exception {
        System.out.println("DownstreamHandlerB.handleDownstream:");
        super.handleDownstream(ctx, e);
    }
}

```


#Client

```java:
package ChannelPipelineTest;

import java.net.InetSocketAddress;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

import org.jboss.netty.bootstrap.ClientBootstrap;
import org.jboss.netty.buffer.ChannelBuffer;
import org.jboss.netty.buffer.ChannelBuffers;
import org.jboss.netty.channel.ChannelFactory;
import org.jboss.netty.channel.ChannelFuture;
import org.jboss.netty.channel.ChannelHandlerContext;
import org.jboss.netty.channel.ChannelPipeline;
import org.jboss.netty.channel.ChannelPipelineFactory;
import org.jboss.netty.channel.Channels;
import org.jboss.netty.channel.ExceptionEvent;
import org.jboss.netty.channel.MessageEvent;
import org.jboss.netty.channel.SimpleChannelUpstreamHandler;
import org.jboss.netty.channel.socket.nio.NioClientSocketChannelFactory;
import org.jboss.netty.handler.codec.string.StringEncoder;

public class Client {
    public static void main(String args[]) {
        ExecutorService bossExecutor = Executors.newCachedThreadPool();
        ExecutorService workerExecutor = Executors.newCachedThreadPool();
        ChannelFactory channelFactory = new NioClientSocketChannelFactory(bossExecutor,
                                                                          workerExecutor);
        ClientBootstrap bootstarp = new ClientBootstrap(channelFactory);
        bootstarp.setPipelineFactory(new AppClientChannelPipelineFactory());

        ChannelFuture future = bootstarp
                .connect(new InetSocketAddress("localhost", 8887));
        future.awaitUninterruptibly();
        if (future.isSuccess()) {
            String msg = "hello word";
            ChannelBuffer buffer = ChannelBuffers.buffer(msg.length());
            buffer.writeBytes(msg.getBytes());
            future.getChannel().write(buffer);
        }
    }
}

class AppClientChannelPipelineFactory implements ChannelPipelineFactory {

    public ChannelPipeline getPipeline() throws Exception {

        ChannelPipeline pipeline = Channels.pipeline();

        pipeline.addLast("encode", new StringEncoder());

        pipeline.addLast("handler", new AppStoreClientHandler());
        return pipeline;

    }

}

class AppStoreClientHandler extends SimpleChannelUpstreamHandler {

    // private static Logger log =
    // Logger.getLogger(AppStoreClientHandler.class);

    @Override

    public void messageReceived(ChannelHandlerContext ctx, MessageEvent e)
            throws Exception {

    }

    @Override

    public void exceptionCaught(ChannelHandlerContext ctx, ExceptionEvent e)
            throws Exception {

        // TODO Auto-generated method stub super.exceptionCaught(ctx, e);

    }

}

```


#Console

##Server

```java:
true
UpstreamHandlerA.messageReceived:BigEndianHeapChannelBuffer(ridx=0, widx=10, cap=10)
UpstreamHandlerB.messageReceived：BigEndianHeapChannelBuffer(ridx=0, widx=10, cap=10)
UpstreamHandlerX.messageReceived:BigEndianHeapChannelBuffer(ridx=0, widx=10, cap=10)
DownstreamHandlerB.handleDownstream:
DownstreamHandlerA.handleDownstream
```

Reference: [Analysis](http://www.tuicool.com/articles/eIz6ry)
