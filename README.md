=== Conditional Recipient for Elementor ===

Contributors: ludovicverbeeck

Donate link: https://github.com/djaysan

Tags: elementor, forms, email, conditional, form recipient, email routing

Requires at least: 5.0

Tested up to: 6.5

Requires PHP: 7.4

Stable tag: 1.0.0

License: GPLv2 or later

License URI: http://www.gnu.org/licenses/gpl-2.0.html

Send form submissions to different email addresses based on form field values in Elementor Forms.

== Description ==

Conditional Recipient for Elementor adds intelligent email routing to your Elementor forms, allowing you to send form submissions to different email addresses based on form field values.

Developed by Ludovic Verbeeck, this plugin seamlessly integrates with Elementor Forms to provide dynamic email routing capabilities.

= Features =

* Easy setup within Elementor Form widget

* Route emails based on any form field value

* Multiple conditional rules support

* Works with all Elementor form field types

* Simple and lightweight

= Requirements =

* WordPress 5.0 or later

* Elementor

* Elementor Pro

== Installation ==

1. Upload the plugin files to the `/wp-content/plugins/conditional-recipient-for-elementor` directory, or install the plugin through the WordPress plugins screen directly.

2. Activate the plugin through the 'Plugins' screen in WordPress

3. Edit any form with Elementor

4. Find the new "Conditional Recipient" section in the form widget settings

5. Set up your conditional email routing rules

== Support ==

For support, please visit:
* [GitHub Repository](https://github.com/djaysan)
* [WordPress.org Plugin Support Forum](https://wordpress.org/support/plugin/conditional-recipient-for-elementor/)

== Credits ==

Special thanks to Github user tomslominski (https://github.com/tomslominski)
Developed by tomslominski
Ported by Ludovic Verbeeck
Released on March 17, 2025

== Frequently Asked Questions ==

= How do I set up conditional recipients? =

1. Edit your form with Elementor
2. In the form widget settings, find the "Conditional Recipient" section
3. Enter the Form Field ID you want to base conditions on
4. Add rules matching field values to email addresses

= Does this work with any form field? =

Yes, you can use any form field type as long as you match the exact value that the field will submit.

= What happens if no conditions match? =

The email will be sent to the default recipient specified in the regular Email section of the form settings.

== Screenshots ==

1. Conditional Recipient settings in Elementor form widget
2. Example of setting up conditional rules

== Changelog ==

= 1.0.0 =
* Initial release
= 1.1.0 =
* Added CC'd
* = 1.2.1 =
* Added Multiple CC'd

== Upgrade Notice ==

= 1.0.0 =
Initial release

If you don't want to install the plugin, just add this code snippet (like WP code Snippet manager):
 ```
<?php
// Exit if accessed directly
if (!defined('ABSPATH')) {
    exit;
}

// Prevent redeclaration
if (!function_exists('namespace_add_conditional_email_controls')) {

    /**
     * Add Conditional Recipient section to settings of Form widget.
     */
    add_action('elementor/element/form/section_form_options/after_section_end', function(\ElementorPro\Modules\Forms\Widgets\Form $widget): void {
        $widget->start_controls_section(
            'section_namespace_conditional_recipient',
            [
                'label' => esc_html__('Conditional Recipient', 'namespace'),
            ]
        );

        $widget->add_control(
            'namespace_conditional_recipient_help',
            [
                'type' => \Elementor\Controls_Manager::RAW_HTML,
                'content_classes' => 'elementor-descriptor',
                'raw' => __('Use this section to determine the email recipient based on submitted form data. Set the Form Field ID to the ID of one of the fields from the Form Fields section, and use the Emails setting to set the recipient emails which correspond to each value of that field.', 'namespace'),
            ]
        );

        $widget->add_control(
            'namespace_conditional_recipient_form_field_id',
            [
                'label' => esc_html__('Form Field ID', 'namespace'),
                'type' => \Elementor\Controls_Manager::TEXT,
                'ai' => ['active' => false],
            ]
        );

        $repeater = new \Elementor\Repeater();

        $repeater->add_control(
            'form_field_value',
            [
                'label' => esc_html__('Form Field Value', 'namespace'),
                'type' => \Elementor\Controls_Manager::TEXT,
                'ai' => ['active' => false],
            ]
        );

        $repeater->add_control(
            'email_address',
            [
                'label' => esc_html__('To Email Address(es)', 'namespace'),
                'type' => \Elementor\Controls_Manager::TEXT,
                'placeholder' => 'to@example.com, another@example.com',
                'description' => esc_html__('Comma-separated list of main recipients.', 'namespace'),
                'ai' => ['active' => false],
            ]
        );

        $repeater->add_control(
            'cc_addresses',
            [
                'label' => esc_html__('CC Email Address(es)', 'namespace'),
                'type' => \Elementor\Controls_Manager::TEXT,
                'placeholder' => 'cc1@example.com, cc2@example.com',
                'description' => esc_html__('Comma-separated list of CC recipients (optional).', 'namespace'),
                'ai' => ['active' => false],
            ]
        );

        $widget->add_control(
            'namespace_conditional_recipient_emails',
            [
                'label' => esc_html__('Emails', 'namespace'),
                'type' => \Elementor\Controls_Manager::REPEATER,
                'fields' => $repeater->get_controls(),
                'title_field' => '{{{ form_field_value }}} → {{{ email_address }}}',
            ]
        );

        $widget->end_controls_section();
    });

    /**
     * Filter the email recipient if Conditional Recipient settings are set.
     */
    add_filter('elementor_pro/forms/record/actions_before', function($record) {
        /** @var \ElementorPro\Modules\Forms\Classes\Form_Record $record */
        $form_field_id = $record->get_form_settings('namespace_conditional_recipient_form_field_id');

        if ($form_field_id) {
            $form_field = $record->get_field(['id' => $form_field_id])[$form_field_id] ?? null;

            if ($form_field) {
                $form_field_value = $form_field['value'];
                $email_settings = $record->get_form_settings('namespace_conditional_recipient_emails');

                foreach ($email_settings as $setting) {
                    if ($form_field_value === $setting['form_field_value'] && $setting['email_address']) {
                        $settings = $record->get('form_settings');

                        // Handle multiple To recipients
                        $to_emails = array_map('sanitize_email', array_map('trim', explode(',', $setting['email_address'])));
                        $to_emails = array_filter($to_emails, 'is_email');

                        if (!empty($to_emails)) {
                            $settings['email_to'] = implode(',', $to_emails);
                        }

                        // Handle CC addresses
                        if (!empty($setting['cc_addresses'])) {
                            $cc_emails = array_map('sanitize_email', array_map('trim', explode(',', $setting['cc_addresses'])));
                            $cc_emails = array_filter($cc_emails, 'is_email');

                            if (!empty($cc_emails)) {
                                $settings['email_to_cc'] = implode(',', $cc_emails);
                            }
                        }

                        $record->set('form_settings', $settings);
                        break;
                    }
                }
            }
        }

        return $record;
    });

    /**
     * Add CC recipients to wp_mail headers.
     */
    add_filter('elementor_pro/forms/wp_mail_args', function($mail_args, $record) {
        $form_settings = $record->get('form_settings');

        if (!empty($form_settings['email_to_cc'])) {
            $cc_addresses = array_map('sanitize_email', array_map('trim', explode(',', $form_settings['email_to_cc'])));
            $cc_addresses = array_filter($cc_addresses, 'is_email');

            if (!empty($cc_addresses)) {
                if (!isset($mail_args['headers']) || !is_array($mail_args['headers'])) {
                    $mail_args['headers'] = [];
                }

                foreach ($cc_addresses as $cc) {
                    $mail_args['headers'][] = 'Cc: ' . $cc;
                }
            }
        }

        return $mail_args;
    }, 10, 2);
}
