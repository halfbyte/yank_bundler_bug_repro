# Test case for a rather obscure bundler bug related to yanked gems

The provided Gemfile and Gemfile lock are sythetic (as in: I modified parts of 
the lockfile from hand) but are analoguous to existing cases we (Depfu) have 
observed in the wild.

## The bug

If the lockfile during a bundler run is in "unlocked" state, bundler resolves
all gems from scratch. This leads to an interesting edge case where a yanked gem
can have dependencies that can not be resolved from the index, as all non-yanked
versions of the gem do not contain said dependencies. 

The compact index implementation looks up all dependencies from all available 
versions and puts them into the index, so if older, yanked versions contain
additional dependencies, these are absent from the index used to do the
resolution.

In this specific case, all old versions of `codecov` that depended on `url` have 
been yanked.

This does lead to confusing error messages as the wrong gem is reported as
yanked. In our case (Depfu) this leads to us being unable to provide a 
resolution for our clients, as we cannot detect the actual gem that's been 
yanked and thus try to update the wrong gem. This may be a very specific usecase
but this does also affect bundler end users as the resolution to this error 
message is obvious.

## How to trigger

If you clone this repo and just run `bundle install`, bundler will give you the
correct error message saying that codecov was yanked:

<code>Your bundle is locked to codecov (0.1.9), but that version could not be found in any of
the sources listed in your Gemfile. If you haven't changed sources, that means the author
of codecov (0.1.9) has removed it. You'll need to update your bundle to a version other
than codecov (0.1.9) that hasn't been removed in order to install.</code>

If you, instead, unlock the bundle (but without unlocking codecov), for example 
by running `bundle update json` you'll get an error message that seems to
suggest that instead of codecov, the url gem was yanked:

<code>Your bundle is locked to url (0.3.2), but that version could not be found in any of the
sources listed in your Gemfile. If you haven't changed sources, that means the author of
url (0.3.2) has removed it. You'll need to update your bundle to a version other than url
(0.3.2) that hasn't been removed in order to install.</code>