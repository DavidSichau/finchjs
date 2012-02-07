# Building Finch requires coffee-script and uglify-js. For
# help installing, try:
#
# `npm -g install coffee-script uglify-js`
#
# Original Cake file from Chosen.js - modified for our use
#   https://github.com/harvesthq/chosen/blob/master/Cakefile
fs              	= require 'fs'
path            	= require 'path'
{spawn, exec}   	= require 'child_process'
CoffeeScript    	= require 'coffee-script'
{parser, uglify}	= require 'uglify-js'


# Get the version number
version_file = 'VERSION'
version = "#{fs.readFileSync(version_file)}".replace /[^0-9a-zA-Z.]*/gm, ''
version_tag = "v#{version}"

sources = [
	"coffee/finch.coffee"
]

destinations = [
	"scripts/finch.js"
]

tests = [
	"tests/tests.coffee"
]

testDestinations = [
	"tests/tests.js"
]

Array::unique = ->
	output = {}
	output[@[key]] = @[key] for key in [0...@length]
	value for key, value of output


# Method used to write a javascript file
write_javascript_file = (filename, body) ->
	fs.writeFileSync filename, """
		/*
			Finch.js - Powerfully simple javascript routing
			by Rick Allen (stoodder) and Greg Smith (smrq)

			Version #{version}
			Full source at https://github.com/stoodder/finchjs
			Copyright (c) 2011 RokkinCat, http://www.rokkincat.com

			MIT License, https://github.com/stoodder/finchjs/blob/master/LICENSE.md
			This file is generated by `cake build`, do not edit it by hand.
		*/
		#{body}
	"""
	console.log "Wrote #{filename}"

# Task to build the current source
task 'build', 'build from source', build = (cb) ->
	file_name = file_contents = null
	try
		code = ""
		minifiedCode = ""
		for source in sources
			file_name = source
			file_contents = "#{fs.readFileSync(source)}"
			code += CoffeeScript.compile(file_contents)
		minifiedCode = parser.parse( code )
		minifiedCode = uglify.ast_mangle( minifiedCode )
		minifiedCode = uglify.ast_squeeze( minifiedCode )
		minifiedCode = uglify.gen_code( minifiedCode )

		for destination in destinations
			write_javascript_file(destination, code)
			write_javascript_file(destination.replace(/\.js$/,'.min.js'), minifiedCode)

		cb() if typeof cb is 'function'
	catch e
		print_error e, file_name, file_contents

task 'build-tests', 'build tests from source', (cb) ->
	file_name = file_contents = null
	try
		code = ""
		for test in tests
			file_name = test
			file_contents = "#{fs.readFileSync(test)}"
			code += CoffeeScript.compile(file_contents)

		for testDestination in testDestinations
			write_javascript_file(testDestination, code)

		cb() if typeof cb is 'function'
	catch e
		print_error e, file_name, file_contents

#Task to watch files (so they're built when saved)
task 'watch', 'watch coffee/ and tests/ for changes and build', ->
	console.log "Watching for changes in coffee/ and tests/"

	for source in sources
		fs.watch( source, (curr, prev) ->
			if +curr.mtime isnt +prev.mtime
				console.log "Saw change in #{source}"
				invoke 'build'
		)
	for test in tests
		fs.watch( test, (curr, prev) ->
			if +curr.mtime isnt +prev.mtime
				console.log "Saw change in #{test}"
				invoke 'build-tests'
		)

run = (cmd, args, cb, err_cb) ->
	exec "#{cmd} #{args.join(' ')}", (err, stdout, stderr) ->
		if err isnt null
			console.error stderr

			if typeof err_cb is 'function'
				err_cb()
			else
				throw "Failed command execution (#{err})."
		else
			cb(stdout) if typeof cb is 'function'

with_clean_repo = (cb) ->
	run 'git', ['diff', '--exit-code'], cb, ->
		throw 'There are files that need to be committed first.'

without_existing_tag = (cb) ->
	run 'git', ['tag'], (stdout) ->
		if stdout.split("\n").indexOf( version_tag ) >= 0
			throw 'This tag has already been committed to the repo.'
		else
			cb()

push_repo = (args=[], cb, cb_err) ->
	run 'git', ['push'].concat(args), cb, cb_err

print_error = (error, file_name, file_contents) ->
	line = error.message.match /line ([0-9]+):/
	if line && line[1] && line = parseInt(line[1])
		contents_lines = file_contents.split "\n"
		first = if line-4 < 0 then 0 else line-4
		last  = if line+3 > contents_lines.size then contents_lines.size else line+3
		console.log "Error compiling #{file_name}. \"#{error.message}\"\n"
		index = 0
		for line in contents_lines[first...last]
			index++
			line_number = first + 1 + index
			console.log "#{(' ' for [0..(3-(line_number.toString().length))]).join('')} #{line}"
	else
		console.log "Error compiling #{file_name}: #{error.message}"

git_commit = (message) ->
	run "git", ["commit", '-a', '-m', message]

git_tag = (cb, cb_err) ->
	run 'git', ['tag', '-a', '-m', "\"Version #{version}\"", version_tag], cb, cb_err

git_untag = (e) ->
	console.log "Failure to tag caught: #{e}"
	console.log "Removing tag #{version_tag}"
	run 'git', ['tag', '-d', version_tag]

task 'major', 'Executing a major version update', () ->

	console.log "Trying to run a major version update"

	v = version.match(/^([0-9]+)\.([0-9]+)\.([0-9]+)$/)
	v[1]++
	v[2] = v[3] = 0
	version = "#{v[1]}.#{v[2]}.#{v[3]}"

	fs.writeFileSync(version_file, version)

	invoke 'build'
	invoke 'build-tests'

	git_commit("\"Updating to Major version #{version}\"")

	git_tag(->)

	console.log "Finished updating major version"


task 'minor', 'Executing a minor version update', () ->

	console.log "Trying to run a minor versino update"

	v = version.match(/^([0-9]+)\.([0-9]+)\.([0-9]+)$/)
	v[2]++
	v[3] = 0
	version = "#{v[1]}.#{v[2]}.#{v[3]}"

	fs.writeFileSync(version_file, version)

	invoke 'build'
	invoke 'build-tests'

	git_commit("\"Updating to Minor version #{version}\"")

	git_tag(->)

	console.log "Finished updating minor version"


task 'patch', 'Executing a patch version update', () ->

	console.log "Trying to run a patch version update"

	v = version.match(/^([0-9]+)\.([0-9]+)\.([0-9]+)$/)
	v[3]++
	version = "#{v[1]}.#{v[2]}.#{v[3]}"

	fs.writeFileSync(version_file, version)

	invoke 'build'
	invoke 'build-tests'

	git_commit("\"Updating to Patch version #{version}\"")

	git_tag(->)
	
	console.log "Finished updating patch version"


task 'release', 'build, tag the current release, and push', ->
	console.log "Trying to tag #{version_tag}..."
	with_clean_repo( ->
		without_existing_tag( ->
			build( ->
				git_tag ( ->
					push_repo [], ( ->
						push_repo ['--tags'], ( ->
							console.log "Successfully tagged #{version_tag}: https://github.com/stoodder/finchjs/tree/#{version_tag}"

						), git_untag
					), git_untag
				), git_untag
			)
		)
	)
