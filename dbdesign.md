erDiagram
    %% ========================================
    %% USER MANAGEMENT
    %% ========================================
    auth_users ||--|| profiles : "extends (1:1)"
    profiles ||--o| customers : "is-a (1:1)"
    profiles ||--o| drivers : "is-a (1:1)"
    profiles ||--|| wallets : "has (1:1 auto-created)"
    
    %% ========================================
    %% RIDE FLOW
    %% ========================================
    customers ||--o{ rides : "books (1:many)"
    drivers ||--o{ rides : "accepts (1:many)"
    rides ||--o{ ride_status_history : "logs (1:many)"
    rides ||--o{ ratings : "receives (1:many)"
    
    %% ========================================
    %% PAYMENT SYSTEM
    %% ========================================
    wallets ||--o{ transactions : "contains (1:many)"
    
    %% ========================================
    %% COMMUNICATION
    %% ========================================
    profiles ||--o{ messages : "sends (1:many)"
    profiles ||--o{ messages : "receives (1:many)"
    profiles ||--o{ notifications : "receives (1:many)"
    rides ||--o{ messages : "contains (1:many)"
    rides ||--o{ notifications : "triggers (1:many)"
    
    %% ========================================
    %% RATINGS (Additional relationships)
    %% ========================================
    profiles ||--o{ ratings : "gives rating (1:many)"
    profiles ||--o{ ratings : "receives rating (1:many)"
    
    %% ========================================
    %% OPERATIONS
    %% ========================================
    drivers ||--o{ driver_schedules : "sets (1:many)"
    
    %% ========================================
    %% ADDITIONAL RELATIONSHIPS
    %% ========================================
    profiles ||--o{ rides : "can cancel (1:many)"
    pricing_config ||--o{ rides : "calculates fare (1:many)"

    %% ========================================
    %% TABLE DEFINITIONS
    %% ========================================

    auth_users {
        UUID id PK "Supabase Auth"
        VARCHAR email UK
        TEXT encrypted_password
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    profiles {
        UUID id PK "FK to auth_users"
        VARCHAR email UK "NOT NULL"
        VARCHAR phone
        VARCHAR full_name "NOT NULL"
        TEXT profile_picture_url
        ENUM user_type "customer|driver|manager"
        BOOLEAN is_active "DEFAULT true"
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    customers {
        UUID id PK "FK to profiles"
        VARCHAR student_id
        VARCHAR university_affiliation
        VARCHAR emergency_contact_name
        VARCHAR emergency_contact_phone
        VARCHAR preferred_payment_method
        INTEGER total_rides_taken "DEFAULT 0"
        TIMESTAMP created_at
    }

    drivers {
        UUID id PK "FK to profiles"
        VARCHAR license_number UK "NOT NULL"
        DATE license_expiry "NOT NULL"
        VARCHAR vehicle_make
        VARCHAR vehicle_model
        INTEGER vehicle_year
        VARCHAR vehicle_plate UK
        VARCHAR vehicle_color
        INTEGER max_passengers "DEFAULT 4"
        BOOLEAN is_verified "DEFAULT false"
        BOOLEAN is_available "DEFAULT false"
        DECIMAL current_location_lat "For geolocation"
        DECIMAL current_location_lng "For geolocation"
        DECIMAL rating "DEFAULT 5.00"
        INTEGER total_rides_completed "DEFAULT 0"
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    wallets {
        UUID id PK
        UUID user_id FK "UNIQUE to profiles"
        DECIMAL balance "DEFAULT 0.00"
        VARCHAR currency "DEFAULT USD"
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    transactions {
        UUID id PK
        UUID wallet_id FK "NOT NULL"
        VARCHAR transaction_type "topup|ride_payment|refund|withdrawal"
        DECIMAL amount "NOT NULL"
        DECIMAL balance_before
        DECIMAL balance_after
        VARCHAR stripe_payment_id "Stripe integration"
        TEXT description
        VARCHAR status "pending|completed|failed|refunded"
        JSONB metadata
        TIMESTAMP created_at
    }

    rides {
        UUID id PK
        UUID customer_id FK "NOT NULL to customers"
        UUID driver_id FK "to drivers"
        TEXT pickup_address "NOT NULL"
        DECIMAL pickup_lat "NOT NULL"
        DECIMAL pickup_lng "NOT NULL"
        TIMESTAMP pickup_time "NOT NULL"
        TEXT dropoff_address "NOT NULL"
        DECIMAL dropoff_lat "NOT NULL"
        DECIMAL dropoff_lng "NOT NULL"
        TIMESTAMP estimated_dropoff_time
        TIMESTAMP actual_dropoff_time
        VARCHAR flight_number "FLIGHT_TRACKING"
        TIMESTAMP flight_arrival_time "FLIGHT_TRACKING"
        VARCHAR flight_status "FLIGHT_TRACKING"
        INTEGER num_passengers "DEFAULT 1"
        DECIMAL estimated_distance_km
        INTEGER estimated_duration_min
        DECIMAL estimated_fare
        DECIMAL actual_fare
        ENUM status "pending|assigned|in_progress|completed|cancelled"
        TEXT cancellation_reason
        UUID cancelled_by FK "to profiles"
        TEXT special_instructions
        TEXT customer_notes
        TEXT driver_notes
        TIMESTAMP assigned_at
        TIMESTAMP started_at
        TIMESTAMP completed_at
        TIMESTAMP cancelled_at
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    ride_status_history {
        UUID id PK
        UUID ride_id FK "NOT NULL to rides"
        ENUM status "ride status value"
        UUID changed_by FK "to profiles"
        TEXT notes
        TIMESTAMP created_at
    }

    ratings {
        UUID id PK
        UUID ride_id FK "NOT NULL to rides"
        UUID rated_by FK "NOT NULL to profiles (giver)"
        UUID rated_user FK "NOT NULL to profiles (receiver)"
        INTEGER rating "1-5 NOT NULL"
        TEXT review_text
        TIMESTAMP created_at
    }

    messages {
        UUID id PK
        UUID ride_id FK "to rides"
        UUID sender_id FK "NOT NULL to profiles"
        UUID recipient_id FK "NOT NULL to profiles"
        TEXT message_text "NOT NULL"
        BOOLEAN is_read "DEFAULT false"
        VARCHAR twilio_message_id "Twilio integration"
        TIMESTAMP created_at
    }

    notifications {
        UUID id PK
        UUID user_id FK "NOT NULL to profiles"
        VARCHAR notification_type "ride_assigned|driver_arrived|etc"
        VARCHAR title "NOT NULL"
        TEXT message "NOT NULL"
        UUID ride_id FK "to rides"
        BOOLEAN is_read "DEFAULT false"
        BOOLEAN is_pushed "DEFAULT false"
        JSONB metadata
        TIMESTAMP created_at
    }

    driver_schedules {
        UUID id PK
        UUID driver_id FK "NOT NULL to drivers"
        TIMESTAMP start_time "NOT NULL"
        TIMESTAMP end_time "NOT NULL"
        BOOLEAN is_available "DEFAULT true"
        TEXT notes
        TIMESTAMP created_at
    }

    pricing_config {
        UUID id PK
        DECIMAL base_fare "DEFAULT 5.00"
        DECIMAL per_km_rate "DEFAULT 2.50"
        DECIMAL per_minute_rate "DEFAULT 0.50"
        DECIMAL minimum_fare "DEFAULT 8.00"
        DECIMAL surge_multiplier "DEFAULT 1.00"
        DECIMAL commission_percentage "DEFAULT 20.00"
        BOOLEAN is_active "DEFAULT true"
        TIMESTAMP effective_from
        TIMESTAMP created_at
    }
