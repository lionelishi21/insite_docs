-- PostgreSQL Tables from ibisrael@yahoo.com
-- All production tables for the iLocate platform.
-- Collected from emails received May 12–19, 2026.

-- ─────────────────────────────────────────────────────────────
-- Table: public.sites
-- ─────────────────────────────────────────────────────────────

CREATE TABLE IF NOT EXISTS public.sites
(
    site_id uuid NOT NULL DEFAULT uuid_generate_v4(),
    site_name text COLLATE pg_catalog."default" NOT NULL,
    site_code character varying(50) COLLATE pg_catalog."default" NOT NULL,
    organization_name text COLLATE pg_catalog."default",
    status character varying(20) COLLATE pg_catalog."default" NOT NULL DEFAULT 'pilot'::character varying,
    timezone character varying(50) COLLATE pg_catalog."default" NOT NULL DEFAULT 'America/Los_Angeles'::character varying,
    address_line1 text COLLATE pg_catalog."default",
    address_line2 text COLLATE pg_catalog."default",
    city text COLLATE pg_catalog."default",
    state character varying(2) COLLATE pg_catalog."default",
    postal_code character varying(10) COLLATE pg_catalog."default",
    country character varying(2) COLLATE pg_catalog."default" DEFAULT 'US'::character varying,
    pilot_start_date date,
    pilot_end_date date,
    created_at timestamp with time zone NOT NULL DEFAULT now(),
    updated_at timestamp with time zone NOT NULL DEFAULT now(),
    auth_type integer,
    hospital_id uuid,
    entra_ad_site_synch_active bit(1),
    admin_primary_name character varying COLLATE pg_catalog."default",
    admin_primary_email character varying COLLATE pg_catalog."default",
    admin_primary_phone character varying COLLATE pg_catalog."default",
    admin_secondary_name character varying COLLATE pg_catalog."default",
    admin_secondary_email character varying COLLATE pg_catalog."default",
    admin_secondary_phone character varying COLLATE pg_catalog."default",
    admin_preferred_contact_type integer,
    ownership_type smallint,
    is_site_active boolean,
    notes text COLLATE pg_catalog."default",
    go_live_date timestamp with time zone,
    subdomain character varying COLLATE pg_catalog."default",
    CONSTRAINT sites_pkey PRIMARY KEY (site_id),
    CONSTRAINT sites_status_check CHECK (status::text = ANY (ARRAY[
        'pilot'::character varying::text,
        'active'::character varying::text,
        'suspended'::character varying::text,
        'terminated'::character varying::text
    ]))
)
TABLESPACE pg_default;

ALTER TABLE IF EXISTS public.sites OWNER TO postgres;

CREATE UNIQUE INDEX IF NOT EXISTS idx_sites_site_code
    ON public.sites USING btree (site_code COLLATE pg_catalog."default" ASC NULLS LAST)
    TABLESPACE pg_default;

CREATE INDEX IF NOT EXISTS idx_sites_status
    ON public.sites USING btree (status COLLATE pg_catalog."default" ASC NULLS LAST)
    TABLESPACE pg_default;

CREATE OR REPLACE TRIGGER trg_sites_updated_at
    BEFORE UPDATE ON public.sites
    FOR EACH ROW EXECUTE FUNCTION public.set_sites_updated_at();


-- ─────────────────────────────────────────────────────────────
-- Table: public.location_type
-- ─────────────────────────────────────────────────────────────

CREATE TABLE IF NOT EXISTS public.location_type
(
    location_type_id uuid NOT NULL DEFAULT uuid_generate_v4(),
    location_type_code character varying(40) COLLATE pg_catalog."default" NOT NULL,
    location_type_name character varying(100) COLLATE pg_catalog."default" NOT NULL,
    supports_availability boolean NOT NULL DEFAULT false,
    implies_in_use boolean NOT NULL DEFAULT false,
    is_active boolean NOT NULL DEFAULT true,
    created_at timestamp with time zone NOT NULL DEFAULT now(),
    updated_at timestamp with time zone NOT NULL DEFAULT now(),
    is_patient_area bit(1),
    CONSTRAINT location_type_pkey PRIMARY KEY (location_type_id),
    CONSTRAINT location_type_location_type_code_key UNIQUE (location_type_code)
)
TABLESPACE pg_default;

ALTER TABLE IF EXISTS public.location_type OWNER TO postgres;


-- ─────────────────────────────────────────────────────────────
-- Table: public.site_configuration_facility_layout
-- ─────────────────────────────────────────────────────────────

CREATE TABLE IF NOT EXISTS public.site_configuration_facility_layout
(
    facility_idx integer NOT NULL DEFAULT nextval('site_facility_layout_facility_layout_idx_seq'::regclass),
    hospital_id integer NOT NULL,
    site_id integer NOT NULL,
    building character varying(100) COLLATE pg_catalog."default" NOT NULL,
    floor_sort integer NOT NULL,
    assigned_to character varying(255) COLLATE pg_catalog."default",
    created_date timestamp without time zone NOT NULL DEFAULT CURRENT_TIMESTAMP,
    is_active bit(1) NOT NULL,
    site character varying COLLATE pg_catalog."default",
    zone character varying COLLATE pg_catalog."default",
    site_name character varying COLLATE pg_catalog."default",
    location_type character varying COLLATE pg_catalog."default",
    CONSTRAINT site_facility_layout_pkey PRIMARY KEY (facility_idx)
)
TABLESPACE pg_default;

ALTER TABLE IF EXISTS public.site_configuration_facility_layout OWNER TO postgres;

CREATE INDEX IF NOT EXISTS idx_site_facility_layout_hospital_id
    ON public.site_configuration_facility_layout USING btree (hospital_id ASC NULLS LAST)
    TABLESPACE pg_default;

CREATE INDEX IF NOT EXISTS idx_site_facility_layout_site_id
    ON public.site_configuration_facility_layout USING btree (site_id ASC NULLS LAST)
    TABLESPACE pg_default;


-- ─────────────────────────────────────────────────────────────
-- Table: public.intake_facility_layout
-- ─────────────────────────────────────────────────────────────

CREATE TABLE IF NOT EXISTS public.intake_facility_layout
(
    location_id uuid NOT NULL DEFAULT uuid_generate_v4(),
    location_name text COLLATE pg_catalog."default" NOT NULL,
    address text COLLATE pg_catalog."default",
    created_at timestamp without time zone DEFAULT now(),
    site_id "char"[],
    floor bigint,         -- the floor level
    building "char"[],    -- name of building where device is located
    wing "char"[],        -- wing where device is located
    status bigint,
    location_code "char"[],
    workorder_id character varying COLLATE pg_catalog."default",
    location_type character varying COLLATE pg_catalog."default",
    proximity character varying COLLATE pg_catalog."default",
    assigned_receiver_signature_id uuid
)
TABLESPACE pg_default;

ALTER TABLE IF EXISTS public.intake_facility_layout OWNER TO postgres;

CREATE OR REPLACE TRIGGER trg_locations_updated_at
    BEFORE UPDATE ON public.intake_facility_layout
    FOR EACH ROW EXECUTE FUNCTION public.set_locations_updated_at();


-- ─────────────────────────────────────────────────────────────
-- Table: public.site_configuration_receivers
-- ─────────────────────────────────────────────────────────────

CREATE TABLE IF NOT EXISTS public.site_configuration_receivers
(
    receiver_idx integer NOT NULL DEFAULT nextval('workorder_receivers_detail_assignments_receiver_idx_seq'::regclass),
    site_id integer NOT NULL,
    device_name character varying(255) COLLATE pg_catalog."default" NOT NULL,
    signature_id character varying(255) COLLATE pg_catalog."default" NOT NULL,
    location_id integer,
    created_at timestamp without time zone DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamp without time zone DEFAULT CURRENT_TIMESTAMP,
    date_completed date,
    telemetry_interval_seconds integer,
    receiver_assigned_location text COLLATE pg_catalog."default",
    site character varying COLLATE pg_catalog."default",
    building character varying COLLATE pg_catalog."default",
    floor_sort integer,
    zone character varying COLLATE pg_catalog."default",
    is_active bit(1),
    gateway_id uuid,
    location_type character varying COLLATE pg_catalog."default",
    heart_beat_interval_seconds integer,
    proximity_group character varying COLLATE pg_catalog."default",
    serial_number character varying COLLATE pg_catalog."default",
    workorder_id character varying COLLATE pg_catalog."default",
    facility_id integer,
    raw_config_file_json_data text COLLATE pg_catalog."default",
    status character varying COLLATE pg_catalog."default",
    CONSTRAINT workorder_receivers_detail_assignments_pkey PRIMARY KEY (receiver_idx)
)
TABLESPACE pg_default;


-- ─────────────────────────────────────────────────────────────
-- Table: public.site_configuration_asset_tags
-- ─────────────────────────────────────────────────────────────

CREATE TABLE IF NOT EXISTS public.site_configuration_asset_tags
(
    asset_tag_config_idx bigint NOT NULL DEFAULT nextval('workorder_asset_configuration_workorder_config_idx_seq'::regclass),
    created_by character varying(100) COLLATE pg_catalog."default" NOT NULL,
    created_at timestamp with time zone NOT NULL DEFAULT CURRENT_TIMESTAMP,
    asset_tag_complete boolean NOT NULL DEFAULT false,
    work_item_status character varying(50) COLLATE pg_catalog."default" NOT NULL DEFAULT 'PENDING'::character varying,
    date_completed timestamp with time zone,
    asset_type_id uuid,
    is_complete bit(1),
    serial_number character varying COLLATE pg_catalog."default",
    workorder_id character varying COLLATE pg_catalog."default",
    updated_at timestamp with time zone,
    CONSTRAINT workorder_asset_configuration_pkey PRIMARY KEY (asset_tag_config_idx)
)
TABLESPACE pg_default;

ALTER TABLE IF EXISTS public.site_configuration_asset_tags OWNER TO postgres;


-- ─────────────────────────────────────────────────────────────
-- Table: public.asset_tag_signal_events
-- ─────────────────────────────────────────────────────────────

CREATE TABLE IF NOT EXISTS public.asset_tag_signal_events
(
    event_id uuid NOT NULL DEFAULT uuid_generate_v4(),
    site_id uuid NOT NULL,
    tag_id uuid NOT NULL,
    asset_gateway_id uuid NOT NULL,
    rssi integer,
    motion_detected boolean NOT NULL DEFAULT false,
    motion_score integer,
    battery_level integer,
    temperature_c numeric(5,2),
    received_at timestamp with time zone NOT NULL DEFAULT now(),
    device_name character varying COLLATE pg_catalog."default",
    receiver_gateway_id uuid,
    last_motion_at timestamp with time zone,
    CONSTRAINT asset_tag_signal_events_pkey PRIMARY KEY (event_id)
)
TABLESPACE pg_default;

ALTER TABLE IF EXISTS public.asset_tag_signal_events OWNER TO postgres;


-- ─────────────────────────────────────────────────────────────
-- Table: public.asset_search_history
-- ─────────────────────────────────────────────────────────────

CREATE TABLE IF NOT EXISTS public.asset_search_history
(
    search_audit_id uuid NOT NULL DEFAULT uuid_generate_v4(),
    site_id uuid NOT NULL,
    user_id uuid,
    session_id uuid,
    search_timestamp timestamp with time zone NOT NULL DEFAULT now(),
    asset_type_id uuid,
    search_text character varying(255) COLLATE pg_catalog."default",
    search_source character varying(50) COLLATE pg_catalog."default" NOT NULL DEFAULT 'mobile'::character varying,
    selected_current_location character varying(150) COLLATE pg_catalog."default",
    location_zone_id uuid,
    last_seen_filter character varying(50) COLLATE pg_catalog."default",
    exclude_patient_rooms boolean NOT NULL DEFAULT true,
    only_reliable_results boolean NOT NULL DEFAULT true,
    search_success_flag boolean,
    no_results_flag boolean NOT NULL DEFAULT false,
    created_at timestamp with time zone NOT NULL DEFAULT now(),
    CONSTRAINT equipment_search_audit_pkey PRIMARY KEY (search_audit_id)
)
TABLESPACE pg_default;

ALTER TABLE IF EXISTS public.asset_search_history OWNER TO postgres;

CREATE INDEX IF NOT EXISTS idx_equipment_search_audit_asset_type
    ON public.asset_search_history USING btree (asset_type_id ASC NULLS LAST)
    TABLESPACE pg_default;

CREATE INDEX IF NOT EXISTS idx_equipment_search_audit_site
    ON public.asset_search_history USING btree (site_id ASC NULLS LAST)
    TABLESPACE pg_default;

CREATE INDEX IF NOT EXISTS idx_equipment_search_audit_timestamp
    ON public.asset_search_history USING btree (search_timestamp ASC NULLS LAST)
    TABLESPACE pg_default;

CREATE INDEX IF NOT EXISTS idx_equipment_search_audit_user
    ON public.asset_search_history USING btree (user_id ASC NULLS LAST)
    TABLESPACE pg_default;


-- ─────────────────────────────────────────────────────────────
-- Table: public.equipment_search_audit
-- ─────────────────────────────────────────────────────────────

CREATE TABLE IF NOT EXISTS public.equipment_search_audit
(
    search_audit_id uuid NOT NULL DEFAULT uuid_generate_v4(),
    site_id uuid NOT NULL,
    user_id uuid,
    session_id uuid,
    search_timestamp timestamp with time zone NOT NULL DEFAULT now(),
    asset_type_id uuid,
    search_text character varying(255) COLLATE pg_catalog."default",
    search_source character varying(50) COLLATE pg_catalog."default" NOT NULL DEFAULT 'mobile'::character varying,
    selected_current_location character varying(150) COLLATE pg_catalog."default",
    location_zone_id uuid,
    last_seen_filter character varying(50) COLLATE pg_catalog."default",
    exclude_patient_rooms boolean NOT NULL DEFAULT true,
    only_reliable_results boolean NOT NULL DEFAULT true,
    search_success_flag boolean,
    no_results_flag boolean NOT NULL DEFAULT false,
    created_at timestamp with time zone NOT NULL DEFAULT now(),
    CONSTRAINT equipment_search_audit_pkey PRIMARY KEY (search_audit_id)
)
TABLESPACE pg_default;


-- ─────────────────────────────────────────────────────────────
-- Table: public.asset_manual_status_history
-- ─────────────────────────────────────────────────────────────
-- Tracks nurses manually setting an asset status.
-- Valid new_status_code values:
--   AVAILABLE | IN_USE | NEEDS_CLEANING | UNDER_MAINTENANCE | MARKED_MISSING
--
-- asset_status_current possible statuses (sensor event stream):
--   UNKNOWN | MOVING | RECENTLY_MOVED | STATIONARY | PROBABLY_AVAILABLE
--   PROBABLY_IN_USE | LOST_SIGNAL | MAINTENANCE_HOLD | LOW_BATTERY
--   RESERVED | OUT_OF_SERVICE | NEEDS_CLEANING | MARKED_MISSING

CREATE TABLE IF NOT EXISTS public.asset_manual_status_history
(
    manual_status_history_id uuid NOT NULL DEFAULT uuid_generate_v4(),
    asset_id uuid NOT NULL,
    site_id uuid,
    nurse_user_id uuid,
    previous_status_code character varying(40) COLLATE pg_catalog."default",
    new_status_code character varying(40) COLLATE pg_catalog."default" NOT NULL,
    status_reason text COLLATE pg_catalog."default",
    notes text COLLATE pg_catalog."default",
    changed_by_name character varying(150) COLLATE pg_catalog."default",
    changed_by_email character varying(255) COLLATE pg_catalog."default",
    created_at timestamp with time zone NOT NULL DEFAULT now(),
    CONSTRAINT asset_manual_status_history_pkey PRIMARY KEY (manual_status_history_id),
    CONSTRAINT chk_manual_asset_status_code CHECK (new_status_code::text = ANY (ARRAY[
        'AVAILABLE'::character varying,
        'IN_USE'::character varying,
        'NEEDS_CLEANING'::character varying,
        'UNDER_MAINTENANCE'::character varying,
        'MARKED_MISSING'::character varying
    ]::text[]))
)
TABLESPACE pg_default;
