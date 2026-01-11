  # Kafka Analytics Pipeline                                                                                                                
                                                                                                                                            
  Real-time analytics pipeline using Apache Kafka, Quixstreams stream processing, and TimescaleDB for time-series data storage.             
                                                                                                                                            
  ## Architecture                                                                                                                           
                                                                                                                                            
  API Gateway → Kafka KRaft Cluster → Quixstreams Processors → TimescaleDB                                                                  
       ↓              ↓                        ↓                    ↓                                                                       
    (EOS + Avro)  (3 Brokers)         (Real-time Analytics)   (Time-series)                                                                 
                                                                                                                                            
  ## Key Features                                                                                                                           
                                                                                                                                            
  - **Kafka 4.1 KRaft Mode**: No ZooKeeper dependency, modern Kafka architecture                                                            
  - **Schema Registry**: Avro schema management with backward compatibility                                                                 
  - **Exactly-Once Semantics (EOS)**: Guaranteed data integrity with transactional producers                                                
  - **Quixstreams Processing**: Modern stream processing (2-3x faster than Faust)                                                           
  - **TimescaleDB**: Efficient time-series storage with automatic partitioning                                                              
                                                                                                                                            
  ## Components                                                                                                                             
                                                                                                                                            
  ### API Gateway (Port 8001)                                                                                                               
  - FastAPI application with EOS-enabled Kafka producer                                                                                     
  - Avro serialization with Schema Registry integration                                                                                     
  - Endpoints: `/api/v1/orders`, `/api/v1/product-views`                                                                                    
                                                                                                                                            
  ### Kafka Cluster                                                                                                                         
  - 3 brokers with replication factor 3                                                                                                     
  - Controller quorum for KRaft consensus                                                                                                   
  - Topics: `order_events`, `product_views`, `order_aggregates`, `trend_alerts`                                                             
                                                                                                                                            
  ### Stream Processors (Quixstreams)                                                                                                       
  - **Event Aggregator**: Tumbling window aggregations (5-min windows)                                                                      
  - **Trend Detector**: Anomaly detection with EMA baseline tracking                                                                        
  - **ETL Processor**: Batch loading to TimescaleDB with upsert pattern                                                                     
                                                                                                                                            
  ### TimescaleDB                                                                                                                           
  - Hypertables for automatic time-partitioning                                                                                             
  - Continuous aggregates for pre-computed hourly stats                                                                                     
  - Compression (7-day) and retention (90-day) policies                                                                                     
                                                                                                                                            
  ### Monitoring                                                                                                                            
  - Kafka UI (Port 8080): Cluster management                                                                                                
  - Prometheus (Port 9090): Metrics collection                                                                                              
  - Grafana (Port 3000): Visualization dashboards                                                                                           
                                                                                                                                            
  ## Quick Start                                                                                                                            
                                                                                                                                            
  ```bash                                                                                                                                   
  # Start infrastructure                                                                                                                    
  docker compose up -d kafka-broker-1 kafka-broker-2 kafka-broker-3 schema-registry timescaledb                                             
                                                                                                                                            
  # Wait 30 seconds for cluster initialization                                                                                              
  sleep 30                                                                                                                                  
                                                                                                                                            
  # Start stream processors                                                                                                                 
  docker compose up -d event-aggregator trend-detector etl-processor                                                                        
                                                                                                                                            
  # Start API Gateway                                                                                                                       
  docker compose up -d api-gateway                                                                                                          
                                                                                                                                            
  # Verify services                                                                                                                         
  docker compose ps                                                                                                                         
                                                                                                                                            
  # Create test order                                                                                                                       
  curl -X POST http://localhost:8001/api/v1/orders \                                                                                        
    -H "Content-Type: application/json" \                                                                                                   
    -d '{                                                                                                                                   
      "user_id": "user_001",                                                                                                                
      "items": [                                                                                                                            
        {                                                                                                                                   
          "product_id": "prod_123",                                                                                                         
          "quantity": 2,                                                                                                                    
          "price": 99.99                                                                                                                    
        }                                                                                                                                   
      ]                                                                                                                                     
    }'                                                                                                                                      
                                                                                                                                            
  # Access monitoring                                                                                                                       
  open http://localhost:8080  # Kafka UI                                                                                                    
  open http://localhost:8001/docs  # API Gateway docs                                                                                       
                                                                                                                                            
  Technology Stack                                                                                                                          
  ┌───────────────────┬───────────────────────────────┬───────────────┐                                                                     
  │     Component     │          Technology           │    Version    │                                                                     
  ├───────────────────┼───────────────────────────────┼───────────────┤                                                                     
  │ Message Broker    │ Apache Kafka (KRaft)          │ 7.6.0         │                                                                     
  ├───────────────────┼───────────────────────────────┼───────────────┤                                                                     
  │ Schema Registry   │ Confluent Schema Registry     │ 7.6.0         │                                                                     
  ├───────────────────┼───────────────────────────────┼───────────────┤                                                                     
  │ Stream Processing │ Quixstreams                   │ 2.6.0         │                                                                     
  ├───────────────────┼───────────────────────────────┼───────────────┤                                                                     
  │ Time-Series DB    │ TimescaleDB                   │ PostgreSQL 15 │                                                                     
  ├───────────────────┼───────────────────────────────┼───────────────┤                                                                     
  │ API Framework     │ FastAPI                       │ 0.104.1       │                                                                     
  ├───────────────────┼───────────────────────────────┼───────────────┤                                                                     
  │ Serialization     │ Apache Avro                   │ -             │                                                                     
  ├───────────────────┼───────────────────────────────┼───────────────┤                                                                     
  │ Monitoring        │ Kafka UI, Prometheus, Grafana │ Latest        │                                                                     
  └───────────────────┴───────────────────────────────┴───────────────┘                                                                     
  Project Structure                                                                                                                         
                                                                                                                                            
  kafka-analytics-pipeline/                                                                                                                 
  ├── api_gateway/                # HTTP API with EOS producer                                                                              
  │   ├── app/                                                                                                                              
  │   │   ├── main.py            # FastAPI application                                                                                      
  │   │   ├── models.py          # Pydantic models                                                                                          
  │   │   ├── config.py          # Configuration                                                                                            
  │   │   └── kafka/                                                                                                                        
  │   │       ├── producer.py    # Transactional producer                                                                                   
  │   │       └── serializers.py # Avro serializers                                                                                         
  │   ├── Dockerfile                                                                                                                        
  │   └── requirements.txt                                                                                                                  
  │                                                                                                                                         
  ├── stream_processors/                                                                                                                    
  │   ├── event_aggregator/      # Windowed aggregations                                                                                    
  │   ├── trend_detector/        # Anomaly detection                                                                                        
  │   └── etl_processor/         # Database loading                                                                                         
  │                                                                                                                                         
  ├── shared/                                                                                                                               
  │   ├── schemas/                                                                                                                          
  │   │   ├── avro_schemas.py   # Avro schema definitions                                                                                   
  │   │   └── events.py         # Pydantic models                                                                                           
  │   └── kafka_config/                                                                                                                     
  │       └── config.py         # Kafka configuration                                                                                       
  │                                                                                                                                         
  ├── scripts/                                                                                                                              
  │   └── init-timescale.sql    # Database initialization                                                                                   
  │                                                                                                                                         
  ├── monitoring/                # Prometheus & Grafana configs                                                                             
  ├── load_test/                 # Locust load tests                                                                                        
  └── docker-compose.yml         # Infrastructure orchestration                                                                             
                                                                                                                                            
  Key Concepts                                                                                                                              
                                                                                                                                            
  Exactly-Once Semantics (EOS)                                                                                                              
                                                                                                                                            
  # API Gateway uses transactional producer                                                                                                 
  producer_config = {                                                                                                                       
      'enable.idempotence': True,      # Prevents duplicates                                                                                
      'transactional.id': 'api-gateway',  # Enables transactions                                                                            
      'acks': 'all'                    # Wait for all replicas                                                                              
  }                                                                                                                                         
                                                                                                                                            
  # Stream processors use read_committed isolation                                                                                          
  consumer_config = {                                                                                                                       
      'isolation.level': 'read_committed'  # Only read committed messages                                                                   
  }                                                                                                                                         
                                                                                                                                            
  Windowed Aggregation                                                                                                                      
                                                                                                                                            
  # 5-minute tumbling windows                                                                                                               
  sdf = app.dataframe(input_topic)                                                                                                          
  sdf = sdf.tumbling_window(duration_ms=300000)  # 5 minutes                                                                                
  sdf = sdf.reduce(aggregate_orders).final()                                                                                                
                                                                                                                                            
  Anomaly Detection                                                                                                                         
                                                                                                                                            
  # EMA baseline tracking                                                                                                                   
  new_baseline = alpha * current + (1 - alpha) * old_baseline                                                                               
  if current > baseline * 2:  # 2x spike threshold                                                                                          
      send_alert(severity="HIGH")                                                                                                           
                                                                                                                                            
  Architecture Decisions                                                                                                                    
                                                                                                                                            
  Why Quixstreams over Faust?                                                                                                               
                                                                                                                                            
  - Performance: 2-3x faster processing                                                                                                     
  - Modern: Active development (2023+) vs deprecated Faust                                                                                  
  - Simplicity: Cleaner API, better documentation                                                                                           
                                                                                                                                            
  Why KRaft over ZooKeeper?                                                                                                                 
                                                                                                                                            
  - Future-proof: ZooKeeper removed in Kafka 4.0+                                                                                           
  - Simpler: No separate coordination service                                                                                               
  - Scalable: Better metadata management                                                                                                    
                                                                                                                                            
  Why Avro over JSON?                                                                                                                       
                                                                                                                                            
  - Schema Evolution: Backward/forward compatibility                                                                                        
  - Type Safety: Enforced at serialization time                                                                                             
  - Efficiency: Binary format (smaller message size)                                                                                        
                                                                                                                                            
  Why TimescaleDB over PostgreSQL?                                                                                                          
                                                                                                                                            
  - Auto-partitioning: Time-based chunking                                                                                                  
  - Compression: 10x storage reduction                                                                                                      
  - Continuous Aggregates: Pre-computed views                                                                                               
                                                                                                                                            
  Testing                                                                                                                                   
                                                                                                                                            
  Load Testing                                                                                                                              
                                                                                                                                            
  # Install Locust                                                                                                                          
  pip install locust                                                                                                                        
                                                                                                                                            
  # Run load test (100 users, 60 seconds)                                                                                                   
  cd load_test                                                                                                                              
  locust -f locustfile.py --host=http://localhost:8001 \                                                                                    
    --users 100 --spawn-rate 10 --run-time 60s --headless                                                                                   
                                                                                                                                            
  Manual Testing                                                                                                                            
                                                                                                                                            
  # Create 100 orders                                                                                                                       
  for i in {1..100}; do                                                                                                                     
    curl -X POST http://localhost:8001/api/v1/orders \                                                                                      
      -H "Content-Type: application/json" \                                                                                                 
      -d "{\"user_id\":\"user_$i\",\"items\":[{\"product_id\":\"prod_1\",\"quantity\":1,\"price\":50}]}"                                    
  done                                                                                                                                      
                                                                                                                                            
  # Check aggregates (wait 5 min for window)                                                                                                
  docker exec timescaledb psql -U postgres -d analytics_db \                                                                                
    -c "SELECT * FROM order_aggregates ORDER BY time DESC LIMIT 5;"                                                                         
                                                                                                                                            
  Troubleshooting                                                                                                                           
                                                                                                                                            
  Kafka cluster wont start                                                                                                                 
                                                                                                                                            
  # Clean volumes and restart                                                                                                               
  docker compose down -v                                                                                                                    
  docker compose up -d kafka-broker-1 kafka-broker-2 kafka-broker-3                                                                         
                                                                                                                                            
  Stream processor errors                                                                                                                   
                                                                                                                                            
  # Check logs                                                                                                                              
  docker compose logs event-aggregator --tail=50                                                                                            
                                                                                                                                            
  # Check consumer lag                                                                                                                      
  docker exec kafka-broker-1 kafka-consumer-groups \                                                                                        
    --bootstrap-server localhost:9092 \                                                                                                     
    --describe --group event-aggregator-quix                                                                                                
                                                                                                                                            
  TimescaleDB connection issues                                                                                                             
                                                                                                                                            
  # Verify database                                                                                                                         
  docker exec timescaledb psql -U postgres -d analytics_db -c "\dt"                                                                         
                                                                                                                                            
  # Check init script ran                                                                                                                   
  docker compose logs timescaledb | grep "init.sql"                                                                                         
                                                                                                                                            
  Monitoring & Observability                                                                                                                
                                                                                                                                            
  - Kafka UI: http://localhost:8080 - Topics, consumers, schema registry                                                                    
  - API Docs: http://localhost:8001/docs - Interactive API documentation                                                                    
  - Grafana: http://localhost:3000 - Metrics dashboards (admin/admin)                                                                       
  - Prometheus: http://localhost:9090 - Raw metrics                                                                                         
                                                                                                                                            
  Performance Expectations                                                                                                                  
                                                                                                                                            
  - Throughput: 500-1000+ events/second                                                                                                     
  - Latency: <100ms p95 (API → Kafka)                                                                                                       
  - Processing Lag: <1 second (Kafka → TimescaleDB)                                                                                         
  - EOS Guarantee: Zero duplicates under load                                                                                               
                                                                                                                                            
  Next Steps                                                                                                                                
                                                                                                                                            
  1. Review Avro schemas in shared/schemas/avro_schemas.py                                                                                  
  2. Explore API Gateway EOS implementation in api_gateway/app/kafka/producer.py                                                            
  3. Study Quixstreams processors in stream_processors/*/processors/                                                                        
  4. Query TimescaleDB hypertables and continuous aggregates                                                                                
  5. Run load tests to validate EOS and performance                                                                                         
                                                                                                                                            
  ---                                                                                                                                       
  Project Status: ✅ Production-ready architecture                                                                                          
  Complexity Level: Advanced                                                                                                                
  Key Learning: Kafka EOS, Stream Processing, Time-Series Analytics                                                                         
