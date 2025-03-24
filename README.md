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

== Upgrade Notice ==

= 1.0.0 =
Initial release

If you don't want to install the plugin, just add this code snippet (like WP code Snippet manager):
 ```/**
 * Add Conditional Recipient section to settings of Form widget.
 */
add_action( 'elementor/element/form/section_form_options/after_section_end', function( \ElementorPro\Modules\Forms\Widgets\Form $widget ): void {
	$widget->start_controls_section(
		'section_namespace_conditional_recipient',
		[
			'label' => esc_html__( 'Conditional Recipient', 'namespace' ),
		]
	);

	$widget->add_control(
		'namespace_conditional_recipient_help',
		[
			'type' => \Elementor\Controls_Manager::RAW_HTML,
			'content_classes' => 'elementor-descriptor',
			'raw' => __( 'Use this section to determine the email recipient based on submitted form data. Set the Form Field ID to the ID of one of the fields from the Form Fields section, and use the Emails setting to set the recipient emails which correspond to each value of that field.<br><br>If the options do not match, the email will be sent to the usual recipient sent in the Email section.', 'namespace' ),
		]
	);
	
	$widget->add_control(
		'namespace_conditional_recipient_form_field_id',
		[
			'label' => esc_html__( 'Form Field ID', 'namespace' ),
			'type' => \Elementor\Controls_Manager::TEXT,
			'ai' => [
				'active' => false,
			],
		]
	);

	$repeater = new \Elementor\Repeater();

	$repeater->add_control(
		'form_field_value',
		[
			'label' => esc_html__( 'Form Field Value', 'namespace' ),
			'type' => \Elementor\Controls_Manager::TEXT,
			'ai' => [
				'active' => false,
			],
		]
	);

	$repeater->add_control(
		'email_address',
		[
			'label' => esc_html__( 'Email Address', 'namespace' ),
			'type' => \Elementor\Controls_Manager::TEXT,
			'ai' => [
				'active' => false,
			],
		]
	);
	
	$widget->add_control(
		'namespace_conditional_recipient_emails',
		[
			'label' => esc_html__( 'Emails', 'namespace' ),
			'type' => \Elementor\Controls_Manager::REPEATER,
			'fields' => $repeater->get_controls(),
			'title_field' => '{{{ form_field_value }}} &rarr; {{{ email_address }}}',
		]
	);

	$widget->end_controls_section();
} );

/**
 * Filter the email recipient if Conditional Recipient settings are set.
 */
add_filter( 'elementor_pro/forms/record/actions_before', function( $record ) {
	/** @var \ElementorPro\Modules\Forms\Classes\Form_Record $record */
	$form_field_id = $record->get_form_settings( 'namespace_conditional_recipient_form_field_id' );

	if( $form_field_id ) {
		$form_field = $record->get_field( ['id' => $form_field_id] )[$form_field_id] ?? null;

		if( $form_field ) {
			$form_field_value = $form_field['value'];
			$email_settings = $record->get_form_settings( 'namespace_conditional_recipient_emails' );

			foreach( $email_settings as $setting ) {
				if( $form_field_value === $setting['form_field_value'] && $setting['email_address'] ) {
					$settings = $record->get( 'form_settings' );
					$settings['email_to'] = sanitize_email( $setting['email_address'] );
					$record->set( 'form_settings', $settings );
				}
			}
		}
	}

	return $record;
} );
