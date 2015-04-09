perl-database-sophia
====================

Perl XS for Sophia DB (http://sphia.org/)

# INSTALLATION

```sh
perl Makefile.PL
make
make test
make install
```

# SYNOPSIS

```perl
 use Database::Sophia;
 
 my $env = Database::Sophia->sp_env();
 
 my $err = $env->sp_ctl(SPDIR, SPO_CREAT|SPO_RDWR, "./db");
 die $env->sp_error() if $err == -1;
 
 my $db = $env->sp_open();
 die $env->sp_error() unless $db;
 
 $err = $db->sp_set("login", "lastmac");
 print $db->sp_error(), "\n" if $err == -1;
 
 my $value = $db->sp_get("login", $err);
 
 if($err == -1) {
 	print $db->sp_error(), "\n";
 }
 elsif($err == 0) {
 	print "Key not found", "\n";
 }
 elsif($err == 1) {
 	print "Key found", "\n";
 	print "login: ", $value, "\n";
 }
 
 $db->sp_destroy();
 $env->sp_destroy();
```

# DESCRIPTION

It has unique architecture that was created as a result of research and rethinking of primary algorithmic constraints, associated with a getting popular Log-file based data structures, such as LSM-tree.

See http://sphia.org/

This module uses Sophia v1.1. See http://sphia.org/v11.html

# METHODS

### sp_env

create a new environment handle

```perl
 my $env = Database::Sophia->sp_env();
```

### sp_ctl

configurate a database

##### SPDIR

Sets database directory path and it's open flags to use by sp_open().

```perl
 $env->sp_ctl(SPDIR, SPO_CREAT|SPO_RDWR, "./db");
```

**Possible flags are:**
* SPO_RDWR   - open repository in read-write mode (default)
* SPO_RDONLY - open repository in read-only mode
* SPO_CREAT  - create repository if it is not exists.

##### SPCMP

Sets database comparator function to use by database for a key order determination.

```perl
 my $sub_cmp = sub {
	my ($key_a, $key_b, $arg) = @_;
 }
 
 $env->sp_ctl(SPCMP, $sub_cmp, "arg to callback");
```

##### SPPAGE

Sets database max key count in a single page. This option can be tweaked for performance.

```perl
 $env->sp_ctl(SPPAGE, 1024);
```

##### SPGC

Sets flag that garbage collector should be turn on.

```perl
 $env->sp_ctl(SPGC, 1);
```

##### SPGCF

Sets database garbage collector factor value, which is used to determine whether it is time to start gc.

```perl
 $env->sp_ctl(SPGCF, 0.5);
```

##### SPGROW

Sets new database files initial new size and resize factor. This values are used while database extend during merge.

```perl
 $env->sp_ctl(SPGROW, 16 * 1024 * 1024, 2.0);
```

##### SPMERGE

Sets flag that merger thread must be created during sp_open().

```perl
 $env->sp_ctl(SPMERGE, 1);
```

##### SPMERGEWM

Sets database merge watermark value.

```perl
 $env->sp_ctl(SPMERGEWM, 200000);
```

### sp_open

Open or create a database

```perl
 my $db = $env->sp_open();
```

On success, return database object; On error, it returns undef.


### sp_error

Get a string error description

```perl
 $env->sp_error();
```

### sp_destroy

Free any handle

```perl
 $ptr->sp_destroy();
```

### sp_begin

Begin a transaction

```perl
 $db->sp_begin();
```

### sp_commit

Apply a transaction

```perl
 $db->sp_commit();
```

### sp_rollback

Discard a transaction changes

```perl
 $db->sp_rollback();
```

### sp_set

Insert or replace a key-value pair

```perl
 $db->sp_set("key", "value");
```

### sp_get

Find a key in a database

```perl
 my $error;
 $db->sp_get("key", $error);
```

### sp_delete

Delete key from a database

```perl
 $db->sp_delete("key");
```

### sp_cursor

create a database cursor

```perl
 my $cur = $db->sp_cursor(SPGT, "key");
```

**Possible order are:**
* SPGT  - increasing order (skipping the key, if it is equal)
* SPGTE - increasing order (with key)
* SPLT  - decreasing order (skippng the key, if is is equal)
* SPLTE - decreasing order

After a use, cursor handle should be freed by $cur->sp_destroy() function.

### sp_fetch

Iterate a cursor

```perl
 $cur->sp_fetch();
```

### sp_key

Get current key

```perl
 $cur->sp_key()
```

### sp_keysize

```perl
 $cur->sp_keysize()
```

### sp_value

```perl
 $cur->sp_value()
```

### sp_valuesize

```perl
 $cur->sp_valuesize()
```

# Example

### sp_open

```perl
 use Database::Sophia;
 
 my $env = Database::Sophia->sp_env();
 
 my $err = $env->sp_ctl(SPDIR, SPO_CREAT|SPO_RDWR, "./db");
 die $env->sp_error() if $err == -1;
 
 my $db = $env->sp_open();
 die $env->sp_error() unless $db;
```

### sp_error

```perl
 my $db = $env->sp_open();
 die $env->sp_error() unless $db;
```

### sp_destroy

```perl
 $db->sp_destroy();
 $cur->sp_destroy();
 $env->sp_destroy();
```

### sp_begin

```perl
 my $rc = $db->sp_begin();
 print $env->sp_error(), "\n" if $rc == -1;
 
 $rc = $db->sp_set("key", "value");
 print $env->sp_error(), "\n" if $rc == -1;
 
 $rc = $db->sp_commit();
 print $env->sp_error(), "\n" if $rc == -1;
```

### sp_commit

See sp_begin


### sp_rollback

```perl
 my $rc = $db->sp_begin();
 print $env->sp_error(), "\n" if $rc == -1;
 
 $rc = $db->sp_set("key", "value");
 print $env->sp_error(), "\n" if $rc == -1;
 
 $rc = $db->sp_rollback();
 print $env->sp_error(), "\n" if $rc == -1;
```

### sp_set

```perl
 $rc = $db->sp_set("key", "value");
 print $env->sp_error(), "\n" if $rc == -1;
```

### sp_get

```perl
 my $error;
 my $value = $db->sp_get("key", $error);
 
 if($error == -1) {
 	print $db->sp_error(), "\n";
 }
 elsif($error == 0) {
 	print "Key not found", "\n";
 }
 elsif($error == 1) {
 	print "Key found", "\n";
 	print "key: ", $value, "\n";
 }
```

### sp_fetch

```perl
 my $cur = $db->sp_cursor(SPGT, "key");
 
 while($cur->sp_fetch()) {
 	print $cur->sp_key(), ": ", $cur->sp_value();
	print $cur->sp_keysize(), ": ", $cur->sp_valuesize();
 }
 
 $cur->sp_destroy();
```

### sp_key

See sp_fetch


### sp_keysize

See sp_fetch


### sp_value

See sp_fetch


### sp_valuesize

See sp_fetch


# DESTROY

```perl
 undef $obj;
```

Free mem and destroy object.

#CPAN

http://search.cpan.org/~lastmac/

# AUTHOR

Alexander Borisov <lex.borisov@gmail.com>

# COPYRIGHT AND LICENSE

This software is copyright (c) 2014 by Alexander Borisov.

This is free software; you can redistribute it and/or modify it under the same terms as the Perl 5 programming language system itself.

See libsophia license and COPYRIGHT
http://sphia.org/
