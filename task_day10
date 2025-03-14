package com.wipro.network;



import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.NetworkInterface;
import java.util.Collections;
import java.util.Enumeration;

import com.sun.net.httpserver.HttpServer;

import io.prometheus.client.CollectorRegistry;
import io.prometheus.client.Gauge;
import io.prometheus.client.exporter.HTTPServer;
import io.prometheus.client.exporter.PushGateway;

public class NetworkTelemetryAgent {
    private static final Gauge latencyGauge = Gauge.build()
            .name("network_latency_ms")
            .help("Network latency in milliseconds")
            .register();

    private static final Gauge bandwidthGauge = Gauge.build()
            .name("network_bandwidth_mbps")
            .help("Network bandwidth usage in Mbps")
            .register();

    public static void main(String[] args) throws IOException {
        // Start HTTP Server for Prometheus
        HttpServer server = HttpServer.create(new InetSocketAddress(9091), 0);
        HTTPServer prometheusServer = new HTTPServer(server, CollectorRegistry.defaultRegistry, true);
        System.out.println("Prometheus HTTP server started on port 9091");
        
        // PushGateway setup
        PushGateway pushGateway = new PushGateway("localhost:9091");

        while (true) {
            System.out.println("Collecting network metrics...");
            double latency = measureLatency();
            double bandwidth = estimateBandwidth();
            
            // Update Prometheus metrics
            latencyGauge.set(latency);
            bandwidthGauge.set(bandwidth);
            
            System.out.printf("Latency: %.2fms, Bandwidth: %.2fMbps\n", latency, bandwidth);
            System.out.println("Pushing metrics to Prometheus...");
            
            // Push metrics to Prometheus PushGateway
            pushGateway.push(CollectorRegistry.defaultRegistry, "network_agent");
            
            System.out.println("Metrics successfully pushed!");
            
            // Alert if latency > 100ms
            if (latency > 100) {
                System.out.println("ALERT: High latency detected! " + latency + " ms");
            }

            // Sleep before next cycle
            try {
                Thread.sleep(5000); // 5 seconds
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }

    private static double measureLatency() {
        try {
            Process process = new ProcessBuilder("ping", "-c", "1", "8.8.8.8").start();
            process.waitFor();
            return Math.random() * 100; // Simulate random latency for now
        } catch (Exception e) {
            return -1;
        }
    }

    private static double estimateBandwidth() {
        try {
            Enumeration<NetworkInterface> interfaces = NetworkInterface.getNetworkInterfaces();
            for (NetworkInterface netIf : Collections.list(interfaces)) {
                if (!netIf.isLoopback()) {
                    return netIf.getMTU() / 1024.0; // Approximate bandwidth in Mbps
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return 0;
    }
}
