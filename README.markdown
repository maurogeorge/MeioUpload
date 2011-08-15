# MeioUpload 2.0 Behavior Plugin

This behavior provides to upload files in your application, as well as the possibility to translate the error message (originally only in portuguese) and the use of phpThumb as a better thumbnail generator.

You can also use it in No-DB mode, which means that no data for the upload is stored within the database. You can further this by setting `var $useTable = false` in the model, which will both trigger No-DB mode AND allow quick uploads.


## Installation
- Clone from github : in your behaviors directory type `git clone git://github.com/jrbasso/MeioUpload.git plugins/meio_upload`
- Add as a git submodule : in your behaviors directory type `git submodule add git://github.com/jrbasso/MeioUpload.git plugins/meio_upload`
- Download an archive from github and extract it in `plugins/meio_upload`

* If you require thumbnails for image generation, download the latest copy of phpThumb and extract it into your vendors directory. Should end up like: /vendors/phpThumb/{files}. (http://phpthumb.sourceforge.net)

## Usage
In a model that needs uploading, replace the class declaration with something similar to the following:

	<?php
	class Image extends AppModel {
		var $name = 'Image';
		var $actsAs = array(
			'MeioUpload.MeioUpload' => array('filename')
		);
	}
	?>

You also need to specify the fields in your database like so
<pre>
``	CREATE TABLE `images` (``
``		`id` int(8) unsigned NOT NULL auto_increment,``
``		`filename` varchar(255) default NULL,``
``		`dir` varchar(255) default NULL,``
``		`mimetype` varchar(255) NULL,``
``		`filesize` int(11) unsigned default NULL,``
``		`created` datetime default NULL,``
``		`modified` datetime default NULL,``
``		PRIMARY KEY  (`id`)``
``	) ENGINE=MyISAM  DEFAULT CHARSET=utf8;``
</pre>

Create your upload view, make sure it's a multipart/form-data form, and the filename field is of type 'file':

	<?php
		echo $this->Form->create('Image', array('type' => 'file'));
		echo $this->Form->input('filename', array('type' => 'file'));
		echo $this->Form->input('dir', array('type' => 'hidden'));
		echo $this->Form->input('mimetype', array('type' => 'hidden'));
		echo $this->Form->input('filesize', array('type' => 'hidden'));
		echo $this->Form->end('Submit');
	?>
You'll want to include any other fields in your Model as well :)

Make sure your directory (app/webroot/uploads/image/ in this case, image changes to the name of your model) is at least CHMOD 775, also check your php.ini MAX_FILE_SIZE is enough to support the filesizes you are uploading (default in the behavior is 2MB, can be configured)

The behavior code will save the uploaded file's name in the 'filename' field in database, it will not overwrite existing files, instead it will create a new filename based on the original plus a counter. For thumbnails, when the file is uploaded, it will automatically create 3 thumbnail sizes (this can be configured) and prepend the name to the thumbfiles as directories(i.e. app/webroot/uploads/image/image_001.jpg will produced app/webroot/uploads/thumb/small/image/image_001.jpg, app/webroot/uploads/thumb/medium/image/image_001.jpg, app/webroot/uploads/thumb/large/image/image_001.jpg)

### Deleting an uploaded file while keeping the record
Flag the file for deletion by setting `data[Model][filename][remove]` to something non-empty, e.g. `TRUE`. The uploaded file including possible thumbnails will then be deleted together with adherent database fields upon save. Note that the record will be preserved, only the file meta-data columns will be reset.

## Creating thumbnails

### Installation

Download the latest copy of phpThumb and extract it into your vendors directory. Should end up like: /vendors/phpThumb/{files}. (http://phpthumb.sourceforge.net)

### Basic usage

In a model that needs uploading with thumbnails generation, replace the class declaration with something similar to the following:

	<?php
	class Image extends AppModel {
		var $name = 'Image';
		var $actsAs = array(
			'MeioUpload.MeioUpload' => array(
				'filename' => array(
					'thumbsizes' => array(
						'320x90' => array(
							'width' => 320,
							'height' => 90
						)
					)    	            
				)
			)
		);
	}
	?>

in that case we generate one thumb with size of 320x90px. Recalling that on controller and on views the procedure still the same of the default usage.

By default the thumbnails are generated on a dir with the name of the key of our array thumbsizes. In our case the structure created is uploads/image/filename/thumb/320x90.

### Passing params for thumbnails generations

When we are creating thumbnails we are capable of pass params for our thumbnails, in our case we used two params width and height. Lets see these and others.

#### width

How you could imagine it's define the width of our thumbnail, its is defined on pixels.

If not defined the default value are 150.

#### height

Again how you could imagine it's define the height of our thumbnail, its is defined on pixels.

If not defined the default value are 225.

#### forceAspectRatio

Create a  thumbnail with the values specified by "width" and "height" and resizes the original proportionally, it's not cropped, to fit the sizes, if the image is not proporcional the excess will be filled with whitespace. The generated thumbnail image always have the value specified on "width" and "height" and if the image not be proporcional the excess will be filled with whitespace.

We need pass the params of the alignment of our original image is be positioned, the rest of image will be filled with whitespace if the image is not proporcional. The possible values are:

- L = left
- R = right
- T = top
- B = bottom
- C = center
- BL = bottom left
- BR = bottom right
- TL = top left
- TR = top right

See on example:

	<?php
	class Image extends AppModel {
		var $name = 'Image';
		var $actsAs = array(
			'MeioUpload.MeioUpload' => array(
				'filename' => array(
					'thumbsizes' => array(
						'320x90' => array(
							'width' => 320,
							'height' => 90,
							'forceAspectRatio' => 'C',
						)
					)    	            
				)
			)
		);
	}
	?>

On this example create a thumb image for 320x90px and resizes the original proportionally to fit the sizes, if the image is not proporcional the excedent will be filled with whitespace. And how we used the param "C" the image is aligned on the center of the thumb.